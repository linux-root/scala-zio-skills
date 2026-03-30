### **Error Handling**

*   **Rule**: Model expected domain errors using sealed traits or Algebraic Data Types (ADTs).
    *   **Why**: It enables the Scala compiler to perform exhaustive type checking and provides statically-checked documentation of failure modes.
    *   **Example**: `ZIO.fail(InsufficientFunds(required, actual))`
    *   **Anti-example**: `ZIO.fail(new Exception("Insufficient funds"))`

*   **Rule**: Use `orDie` to shift `Throwable` errors to the defect channel when they are unrecoverable.
    *   **Why**: It simplifies type signatures by focusing the error channel on business logic failures while letting catastrophic bugs propagate as fiber failures.
    *   **Example**: `ZIO.attempt(file.readLine).orDie`
    *   **Anti-example**: `val task: ZIO[Any, Throwable, Unit] = ZIO.attempt(println("Critical internal bug"))`

*   **Rule**: Never "swallow" defects; use `tapDefect` for logging and then propagate the failure.
    *   **Why**: Hiding unexpected errors makes debugging impossible; logging at the site of failure provides context before the error propagates to the system edge.
    *   **Example**: `effect.tapDefect(c => ZIO.logErrorCause("Unexpected failure", c))`
    *   **Anti-example**: `effect.catchAllDefect(_ => ZIO.unit)`

### **Composition**

*   **Rule**: Use for-comprehensions for sequential composition of effects.
    *   **Why**: They flatten nested `flatMap` chains into a readable, procedural-like structure that is easier to maintain.
    *   **Example**: `for { user <- getUser; _ <- notify(user) } yield ()`
    *   **Anti-example**: `getUser.flatMap(user => notify(user).map(_ => ()))`

*   **Rule**: Use `zipLeft` (`<*`) or `zipRight` (`*>`) when the result of one effect is not needed.
    *   **Why**: These operators clearly signal that a result is being discarded, reducing boilerplate in `map` or for-comprehensions.
    *   **Example**: `Console.printLine("Starting") *> runTask`
    *   **Anti-example**: `for { _ <- Console.printLine("Starting"); res <- runTask } yield res`

### **Environment and Dependency Injection**

*   **Rule**: Declare service dependencies in the implementation's class constructor, not in the ZIO environment channel.
    *   **Why**: This separates the service's interface (what it does) from its implementation (how it does it), making the code modular and easier to test.
    *   **Example**: `case class LiveUserRepo(db: Database) extends UserRepo`
    *   **Anti-example**: `class LiveUserRepo { def get: ZIO[Database, E, User] = ZIO.service[Database].flatMap(...) }`

*   **Rule**: Use the `Service Pattern`: define a trait, an implementation case class, and a `ZLayer` companion.
    *   **Why**: It provides a standardized structure that leverages ZIO's automatic layer wiring and dependency inference.
    *   **Example**: `trait Repo; case class LiveRepo() extends Repo; object Repo { val layer = ZLayer.derive[LiveRepo] }`
    *   **Anti-example**: Passing manual instances of services directly through deep call stacks.

### **Resource Handling**

*   **Rule**: Use `ZIO.acquireRelease` for any operation requiring allocation and subsequent deallocation.
    *   **Why**: It guarantees the release action will run regardless of success, failure, or interruption, which `ensuring` cannot guarantee if the effect never starts.
    *   **Example**: `ZIO.acquireRelease(openFile)(f => ZIO.succeed(f.close()))`
    *   **Anti-example**: `ZIO.attempt(openFile).flatMap(f => use(f).ensuring(ZIO.attempt(f.close())))`

*   **Rule**: Use `ZIO.scoped` to define the narrowest possible lifetime for a resource.
    *   **Why**: Releasing resources as soon as they are no longer needed prevents memory leaks and ensures system efficiency.
    *   **Example**: `ZIO.scoped { fileResource.flatMap(readData) }`
    *   **Anti-example**: Opening a resource at the start of the app and never closing it until the JVM exits.

### **Concurrency**

*   **Rule**: Use high-level parallel operators (`zipPar`, `foreachPar`) instead of manual `fork`/`join`.
    *   **Why**: These operators automatically handle fiber supervision, error aggregation, and safe interruption of siblings if one fails.
    *   **Example**: `ZIO.foreachPar(urls)(fetch)`
    *   **Anti-example**: `ZIO.foreach(urls)(url => fetch(url).fork).flatMap(ZIO.collectAll)`

*   **Rule**: Use `ZIO.uninterruptibleMask` instead of `ZIO.uninterruptible` when defining custom operators.
    *   **Why**: It allows the operator to protect critical setup/cleanup logic while letting the caller's task remain interruptible via the `restore` function.
    *   **Example**: `ZIO.uninterruptibleMask(restore => setup *> restore(task) *> cleanup)`
    *   **Anti-example**: `setup.uninterruptible *> task.interruptible *> cleanup.uninterruptible`

*   **Rule**: Always `join` or `interrupt` any fiber you manually `fork`.
    *   **Why**: Failing to handle a fiber creates a "leak" where background tasks may continue running unexpectedly after the parent completes.
    *   **Example**: `effect.fork.flatMap(f => f.join.onInterrupt(f.interrupt))`
    *   **Anti-example**: `effect.fork *> ZIO.unit` (without supervision or joining)
