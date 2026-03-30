### **ZIO Abstraction Decision Guide**

This guide provides specific criteria for selecting the appropriate ZIO abstraction based on your concurrency, resource, and architectural requirements.

#### **1. ZLayer (Dependency Injection & Lifecycle)**
*   **When to use it**: Use for managing dependencies between services and when services require effectful initialization or finalization. It is ideal for "wiring" the application at its edge.
*   **When NOT to use it**: Avoid for passing simple, short-lived values or for data that does not have a lifecycle.
*   **Example use case**: Constructing a `GitHubService` that depends on a `Config` and an `HTTPClient`.

#### **2. Ref (Shared Mutable State)**
*   **When to use it**: Use when you need to share a piece of mutable state across multiple fibers and update it atomically via compare-and-swap.
*   **When NOT to use it**: Do not use if you need to coordinate updates across multiple references simultaneously (use `TRef/STM` instead). Use `FiberRef` if the state must be local to a fiber.
*   **Example use case**: A concurrent request counter or a simple in-memory cache.

#### **3. Queue (Work Distribution)**
*   **When to use it**: Use for distributing unique tasks among multiple worker fibers (FIFO) or as a buffer between push-based and pull-based systems.
*   **When NOT to use it**: Avoid when you need to broadcast the same message to all consumers (use `Hub`).
*   **Example use case**: A background worker pool where multiple fibers take tasks from a shared bounded queue.

#### **4. Hub (Broadcasting)**
*   **When to use it**: Use when every subscriber needs to receive a copy of every message published (one-to-many communication).
*   **When NOT to use it**: Do not use for work distribution where only one worker should process each item.
*   **Example use case**: A real-time chat room or a stock tick distribution system.

#### **5. Schedule (Retries & Repetition)**
*   **When to use it**: Use for describing strategies for retrying failed effects or repeating successful ones based on conditions or time intervals.
*   **When NOT to use it**: Avoid for simple, non-timed recursive loops where the logic is entirely internal to the function.
*   **Example use case**: Retrying a database connection with exponential backoff or running a cleanup task every Monday at 9 AM.

#### **6. Fiber (Low-Level Concurrency)**
*   **When to use it**: Use when implementing new concurrency primitives or when you need low-level manual control over a virtual thread's lifecycle.
*   **When NOT to use it**: Default to high-level operators like `zipPar`, `foreachPar`, or `race` whenever possible to avoid manual management.
*   **Example use case**: Forking a background health-check process that outlives its parent via `forkDaemon`.

#### **7. Scope (Resource Management)**
*   **When to use it**: Use for managing the lifetime of composable resources that require finalization, especially when those resources are used across multiple effects.
*   **When NOT to use it**: For simple resources used within a single lexical block, `ZIO.acquireReleaseWith` is often simpler.
*   **Example use case**: Opening multiple files or network connections that must all be closed accurately when an analysis task finishes.

#### **8. Promise (Synchronization)**
*   **When to use it**: Use when one fiber must wait for a result or signal that will be provided exactly once by another fiber.
*   **When NOT to use it**: Do not use for distributing multiple work items (use `Queue`) or if the state needs to change repeatedly (use `Ref`).
*   **Example use case**: An asynchronous cache lookup where multiple fibers wait for the first fiber to complete a computation.

#### **9. Semaphore (Work Limiting)**
*   **When to use it**: Use to limit the number of fibers that can access a specific region of code or resource simultaneously.
*   **When NOT to use it**: Avoid if you need to coordinate complex transactional state (use `STM`).
*   **Example use case**: Limiting an application to a maximum of 10 concurrent outgoing HTTP requests to an external API.

#### **10. ZStream (High-Level Streaming)**
*   **When to use it**: Use when dealing with zero or more (potentially infinite) values produced incrementally over time.
*   **When NOT to use it**: Do not use if the computation always produces exactly one value (use `ZIO`).
*   **Example use case**: Processing a continuous stream of tweets or reading a massive file line-by-line.
