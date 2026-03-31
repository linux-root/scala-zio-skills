### **Typed Errors vs. Defects**
*   **Typed Errors (Checked Failures):** These are anticipated failure scenarios modeled in the `E` type parameter of `ZIO[R, E, A]`. Because they are part of the type signature, the compiler can verify they are handled. 
    *   *Example:* A `UserNotFound` error when querying a database.
*   **Defects (Unchecked Failures):** These are unanticipated bugs or unrecoverable situations like `StackOverflowError` or `ArithmeticException`. They are not represented in the `E` channel but are captured in a `Cause` data structure.
    *   *Example:* An undocumented `IOException` in a basic console application.

### **Core Error Operators**
*   **`mapError`:** Transforms the error type without handling the failure. Use this when you need to unify different error hierarchies (e.g., converting a `DbError` and `ApiError` into a common `AppError`).
*   **`catchAll`:** Recovers from all typed errors by providing a fallback effect. 
    *   *Usage:* `ZIO.readFile("config.json").catchAll(_ => ZIO.succeed(DefaultConfig))`.
*   **`foldZIO`:** The most general handler; it provides separate effects for success and failure cases. It allows you to transform both paths into a new common success type.

### **Narrowing and Converting Errors**
*   **`orDie`:** Converts a `Throwable` error into a defect. Use this when an error is truly unrecoverable at a specific level of your app, simplifying the type signature to `ZIO[R, Nothing, A]`.
*   **`refineToOrDie`:** Keeps one specific error type in the typed channel while killing the fiber for all others.
    *   *Practical Case:* You can recover from `IOException` but want to crash if a `SecurityException` occurs.

### **Either vs. ZIO Error Channel**
*   **When to use the ZIO Channel:** Use the built-in error channel for standard sequential workflows. It enables automatic short-circuiting in `for-comprehensions`—if the first effect fails, the subsequent ones are never executed.
*   **When to use `Either`:** Use `Either` when errors are unrelated and you want to avoid widening the error channel to `Any`. You can "unwrap" an `Either` into the ZIO channel using `.some` or `.absolve` to make standard composition easier.

### **Rules of Thumb**
*   **Model Domain Errors with ADTs:** Use `sealed traits` or `case classes` for expected failures [21, 26.2.1.3, 985]. This allows the compiler to perform **exhaustive type checking**.
*   **Log Defects at the Edge:** Never "swallow" defects early; log them at the application's entry point where you have the most context, then let the fiber die or rethrow.
*   **Narrow Errors Upward:** Low-level utilities should return broad errors (like `IOException`), while higher business layers should refine these into specific domain failures or defects.

### **Anti-patterns**
*   **Overusing `Task`:** Avoid defaulting to `Task[A]` (which fixes the error type to `Throwable`) for business logic. It forces "defensive programming" because you never know exactly what can fail.
*   **Swallowing Defects:** Catching all defects just to prevent a crash hides the root cause of bugs. Use `.tapDefect` for logging instead.
*   **Ignoring Interruption:** Using `.ensuring` for resource release instead of `acquireReleaseWith`. Interruption can happen in the "gap" after acquisition but before `.ensuring` starts, leading to **resource leaks**.
