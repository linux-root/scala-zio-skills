ZIO concurrency is built on **fibers**, which are lightweight, virtual threads managed by the ZIO runtime rather than the operating system. Because fibers are significantly cheaper than OS threads, applications can scale to run millions of them simultaneously.

### **1. Fork and Join: Manual Concurrency**
The most basic way to start a concurrent process is to **fork** an effect.
*   **`fork`**: Calling `.fork` on an effect starts it immediately on a new fiber and returns a `Fiber` object.
*   **`join`**: Calling `.join` on a fiber semantically blocks the current fiber until the forked fiber completes, returning its result. Crucially, this does not block the underlying OS thread.

**Example:**
```scala
for {
  fiber <- doLongWork.fork           // Starts work in background
  _     <- doOtherWork               // Current fiber continues
  result <- fiber.join               // Waits for background work
} yield result
```

### **2. Interruption: Safe Cancellation**
Interruption allows you to stop a fiber when its result is no longer needed (e.g., a timeout or a user closing a connection).
*   **Finalizers**: Unlike standard Java threads, interrupting a ZIO fiber is **safe** because it ensures all finalizers (cleanup logic) are executed.
*   **Timing**: The runtime checks for interruption between individual instructions. It cannot interrupt a single, "monolithic" block of side-effecting code unless specifically wrapped in constructors like `attemptBlockingCancelable`.

### **3. Parallel Operators (Structured Concurrency)**
ZIO provides high-level operators that handle fiber management, error propagation, and interruption automatically. This follows the principle of **structured concurrency**: fibers are scoped to their parents and are interrupted if the parent fails or terminates.

*   **`zipPar`**: Executes two effects in parallel and returns a tuple of both results. If one fails, the other is immediately interrupted.
*   **`foreachPar`**: Executes an effect for every element in a collection in parallel.
*   **`race`**: Runs two effects concurrently and returns the result of the first one to succeed, interrupting the "loser".

**Example:**
```scala
// Fetch multiple URLs in parallel with a concurrency limit of 10
ZIO.foreachParN(10)(urls)(url => httpClient.get(url))
```

### **4. Race Conditions & Shared State**
Concurrency becomes "hard" when fibers share state. ZIO avoids traditional locks in favor of functional primitives:
*   **`Ref`**: A purely functional version of an `AtomicReference`. It provides atomic updates for single variables.
*   **`STM` (Software Transactional Memory)**: Allows you to compose multiple atomic operations into a single transaction. This prevents complex race conditions like the "bank transfer problem" where one account is debited but the other isn't yet credited.

### **Practical Rules & Best Practices**
*   **Prefer High-Level Operators**: Use `foreachPar`, `zipPar`, and `timeout` instead of manual `fork` and `join` whenever possible.
*   **Join or Interrupt**: If you manually `fork` a fiber, ensure you eventually `join` or `interrupt` it to avoid "dangling" work.
*   **Use `ZIO.scoped`**: Use scopes to define exactly how long a fiber or resource should live.
*   **Protect Critical Work**: Use `.uninterruptible` or `uninterruptibleMask` for logic that must complete once it starts, such as complex state updates.

### **Common Mistakes**
*   **Blocking the Executor**: Running synchronous blocking I/O (like legacy JDBC) in a standard `ZIO.attempt`. This can lead to **thread starvation**. Always use `ZIO.attemptBlocking`.
*   **The Interruption Gap**: Using `.ensuring` for resource cleanup. Interruption can happen after acquisition but before `.ensuring` begins, causing a **leak**. Always use `ZIO.acquireRelease`.
*   **Non-Atomic Composition**: Trying to make two separate `Ref` updates consistent without `STM`. Each update is atomic, but the sequence is not.
*   **Monolithic Effects**: Wrapping a giant `while` loop in `ZIO.succeed`. The runtime can't check for interruption inside that loop, making the fiber unresponsive to cancellation.
