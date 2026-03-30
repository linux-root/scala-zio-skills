* **ZIO[R, E, A] (Functional Effect)**
    * **Definition**: An immutable value describing a concurrent workflow that requires environment `R` and eventually yields `A` or fails with `E`.
    * **Role**: Decouples computation description from execution, enabling referential transparency and composability.
    * **Example**: `val effect: ZIO[Any, Nothing, Unit] = ZIO.succeed(println("Action"))`.

* **Type Parameters (R, E, A)**
    * **Definition**: `R` is the contravariant input (environment); `E` is the covariant failure output; `A` is the covariant success output.
    * **Role**: Provides a statically typed model for dependency injection and error tracking at compile time.
    * **Example**: `ZIO[Console, IOException, String]`.

* **Fibers**
    * **Definition**: Lightweight, virtual threads of execution managed by the ZIO runtime.
    * **Role**: Fundamental concurrency unit allowing millions of independent tasks without OS thread overhead.
    * **Example**: `for { f <- effect.fork; a <- f.join } yield a`.

* **Interruption**
    * **Definition**: A mechanism to stop a fiber's execution and run its finalizers.
    * **Role**: Prevents resource leaks by ensuring unnecessary computations are canceled safely.
    * **Example**: `fiber.interrupt`.

* **Environment (ZEnvironment)**
    * **Definition**: A type-indexed map containing service implementations required by an effect.
    * **Role**: Facilitates modularity by allowing effects to "borrow" services provided at the application's edge.
    * **Example**: `ZIO.service[Clock]`.

* **ZLayer**
    * **Definition**: A "recipe" for constructing services from their dependencies, supporting effectful initialization and finalization.
    * **Role**: Automates complex dependency graph wiring into a single, composable value.
    * **Example**: `val layer = ZLayer.fromFunction(MyServiceLive.apply _)`.

* **Scope**
    * **Definition**: A data type representing the lifetime of a resource where finalizers are stored.
    * **Role**: Unifies resource management, allowing developers to extend or close resource lifetimes declaratively.
    * **Example**: `ZIO.scoped { ZIO.acquireRelease(open)(close).flatMap(use) }`.

* **Resource Safety**
    * **Definition**: Guarantees that resources are released regardless of success, failure, or interruption.
    * **Role**: Essential for building resilient applications that do not leak file handles or connections.
    * **Example**: `ZIO.acquireReleaseWith(acquire)(release)(use)`.
