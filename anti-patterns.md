### **1. Overusing `Task` instead of Domain Errors**
*   **Description**: Defaulting to `Task[A]` (type alias for `ZIO[Any, Throwable, A]`) to handle all possible failures in business logic.
*   **Why it is wrong**: `Throwable` is an unsealed type, meaning the compiler cannot verify if all error cases are handled. It conflates **expected domain errors** (like `UserNotFound`) with **defects** (unrecoverable bugs like `StackOverflowError`), leading to defensive programming.
*   **Correct alternative**: Model expected errors using **Sealed Traits or Algebraic Data Types (ADTs)** to enable exhaustive pattern matching by the compiler.
*   **Example**:
    *   *Anti-pattern*: `def getUser(id: Int): Task[User] = ZIO.fail(new Exception("User not found"))`
    *   *Idiomatic*: `def getUser(id: Int): IO[UserError, User] = ZIO.fail(UserNotFound(id))`

### **2. Mixing `Future` with ZIO Incorrectly**
*   **Description**: Creating a `Future` outside of a ZIO effect and then wrapping it using `ZIO.fromFuture`.
*   **Why it is wrong**: A `Future` represents a **running computation**. If created outside ZIO, it begins executing immediately, breaking referential transparency. This prevents ZIO from being able to **retry, delay, or properly interrupt** the work, as the `Future` is already "in flight".
*   **Correct alternative**: Pass a function that creates the `Future` directly into the `ZIO.fromFuture` constructor so that creation is deferred until the effect is run.
*   **Example**:
    *   *Anti-pattern*: `val f = callLegacyApi(); ZIO.fromFuture(_ => f)`
    *   *Idiomatic*: `ZIO.fromFuture(ec => callLegacyApi()(ec))`

### **3. Misusing Environment (Declaring Dependencies in `R`)**
*   **Description**: Declaring a service's dependencies in the environment channel (`R`) of its own trait methods.
*   **Why it is wrong**: It creates a "leaky" interface where the implementation details (what a service needs) are exposed to every caller. It forces the environment type to "bloat" as dependencies ripple through the entire call stack, making the code harder to read and maintain.
*   **Correct alternative**: Use the **Service Pattern**: define the service trait with an `R` of `Any`, and pass dependencies into the **constructor** of the concrete implementation class.
*   **Example**:
    *   *Anti-pattern*: `trait UserService { def get(id: Int): ZIO[Database, E, User] }`
    *   *Idiomatic*: `case class LiveUserService(db: Database) extends UserService { def get(id: Int): IO[E, User] = ... }`

### **4. Unsafe Resource Handling with `ensuring`**
*   **Description**: Using `.ensuring` to release a resource immediately after an acquisition effect.
*   **Why it is wrong**: Interruption can occur in the "gap" after the resource is acquired but before the `use` effect begins. In this specific window, `ensuring` will not trigger because the parent effect never technically "started" its main logic, resulting in a **resource leak**.
*   **Correct alternative**: Use `ZIO.acquireReleaseWith` or `ZIO.acquireRelease` within a `Scope`, which provides ironclad guarantees that the release logic runs if the acquisition completes.
*   **Example**:
    *   *Anti-pattern*: `acquire.flatMap(r => use(r).ensuring(release(r)))`
    *   *Idiomatic*: `ZIO.acquireReleaseWith(acquire)(release)(use)`

### **5. Blocking Operations without Proper Wrappers**
*   **Description**: Wrapping synchronous, blocking I/O (like `scala.io.Source` or legacy JDBC) in standard constructors like `ZIO.attempt` or `ZIO.succeed`.
*   **Why it is wrong**: ZIO’s default executor uses a small, fixed number of threads optimized for CPU-bound work. Clogging these threads with blocking I/O can lead to **thread starvation**, causing the entire application to hang or perform poorly.
*   **Correct alternative**: Use `ZIO.attemptBlocking` or `ZIO.attemptBlockingIO` to shift the work to a dedicated, dynamically-sized blocking thread pool.
*   **Example**:
    *   *Anti-pattern*: `ZIO.attempt(Source.fromURL(url).mkString)`
    *   *Idiomatic*: `ZIO.attemptBlockingIO(Source.fromURL(url).mkString)`
