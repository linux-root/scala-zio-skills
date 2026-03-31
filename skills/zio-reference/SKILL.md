---
name: zio-reference
description: Use when writing ZIO Scala code — implementing services with ZLayer, handling typed errors vs defects, choosing concurrency primitives (Ref, Queue, Hub, STM, Semaphore), avoiding anti-patterns like blocking on ZIO threads, unsafe resource handling with ensuring, or overusing Task for business logic
license: MIT
compatibility: Designed for Scala/ZIO projects
---

# ZIO Coding Reference

You are an expert Scala/ZIO developer. Follow these rules strictly when generating ZIO code.

## Core Concepts

- **ZIO[R, E, A]**: Immutable effect description. `R` = environment (contravariant), `E` = error (covariant), `A` = success (covariant). Effects are descriptions, not running computations.
- **Fibers**: Lightweight virtual threads managed by ZIO runtime. Millions can run concurrently. Fork starts one, join awaits it.
- **ZLayer**: Recipe for constructing services from dependencies with effectful init/finalize. Automates dependency graph wiring.
- **Scope**: Resource lifetime manager. Finalizers run on success, failure, or interruption.
- **Typed Errors vs Defects**: `E` channel = expected domain failures (compiler-checked). Defects = unexpected bugs captured in `Cause`, not in `E`.

> Deep dive: references/core-concepts.md

## Coding Rules

### Error Handling
> Deep dive: references/error-handling-deep-dive.md

- Model domain errors as sealed trait ADTs: `sealed trait UserError; case class NotFound(id: UUID) extends UserError`
- Use `ZIO.fail(NotFound(id))` not `ZIO.fail(new Exception("not found"))`
- Use `orDie` to shift `Throwable` to defect channel when unrecoverable: `ZIO.attempt(expr).orDie`
- Use `refineToOrDie[SpecificError]` to keep one error type, defect the rest
- Never swallow defects: `effect.tapDefect(c => ZIO.logErrorCause("Unexpected", c))` not `effect.catchAllDefect(_ => ZIO.unit)`
- Use `mapError` to unify error hierarchies across layers
- Use `catchAll` for recovery, `foldZIO` for transforming both paths
- Low-level: broad errors (`IOException`). High-level: refined domain ADTs or defects.
- Use `Either` only when errors are unrelated and you want to avoid widening `E` to `Any`

### Composition
- For-comprehensions for sequential composition:
  ```scala
  for { user <- getUser(id); _ <- notify(user) } yield ()
  ```
- `*>` (zipRight) / `<*` (zipLeft) when discarding one result:
  ```scala
  Console.printLine("Starting") *> runTask
  ```

### Dependencies (Service Pattern)
- Trait defines interface (methods return `IO[E, A]` with `R = Any`)
- Case class implements trait, receives dependencies via constructor
- Companion object provides `ZLayer`
```scala
trait Greeter { def sayHello(name: String): UIO[Unit] }

case class GreeterLive(console: Console) extends Greeter {
  def sayHello(name: String): UIO[Unit] = console.printLine(s"Hello, $name!").orDie
}

object Greeter {
  val layer: URLayer[Console, Greeter] = ZLayer.derive[GreeterLive]
}
```

### Resources
- Always use `ZIO.acquireRelease` or `ZIO.acquireReleaseWith`, never `ensuring` for resource cleanup
- Use `ZIO.scoped` for narrowest possible resource lifetime:
  ```scala
  ZIO.scoped { ZIO.acquireRelease(open)(r => ZIO.succeed(r.close())).flatMap(use) }
  ```

### Concurrency
> Deep dive: references/concurrency-and-fiber.md

- Prefer `foreachPar`, `zipPar`, `race` over manual `fork`/`join`
- Always `join` or `interrupt` any manually forked fiber
- Use `ZIO.attemptBlocking` or `ZIO.attemptBlockingIO` for blocking I/O, never `ZIO.attempt`
- Use `ZIO.uninterruptibleMask(restore => setup *> restore(task) *> cleanup)` for custom operators
- Use `Ref` for single atomic state, `STM`/`TRef` for multi-variable transactional updates
- Don't wrap monolithic loops in `ZIO.succeed` — runtime can't interrupt inside them

## Anti-Patterns (Never Do These)
> Deep dive: references/anti-patterns.md

| Anti-Pattern | Correct |
|---|---|
| `Task[A]` for business logic | `IO[DomainError, A]` with sealed trait ADT |
| `val f = api(); ZIO.fromFuture(_ => f)` | `ZIO.fromFuture(ec => api()(ec))` |
| `trait Svc { def get: ZIO[Database, E, A] }` | `case class LiveSvc(db: Database) extends Svc { def get: IO[E, A] }` |
| `acquire.flatMap(r => use(r).ensuring(release(r)))` | `ZIO.acquireReleaseWith(acquire)(release)(use)` |
| `ZIO.attempt(blockingIOCall())` | `ZIO.attemptBlockingIO(blockingIOCall())` |
| `effect.catchAllDefect(_ => ZIO.unit)` | `effect.tapDefect(c => ZIO.logErrorCause("err", c))` |
| `effect.fork *> ZIO.unit` | `effect.fork.flatMap(_.join)` or use structured concurrency |
| `ZIO.succeed(while(true) { ... })` | Recursive ZIO effect or `.forever` |
| `object Svc { def method(a: A): ZIO[Svc, E, B] = ZIO.serviceWithZIO(_.method(a)) }` | Call `ZIO.serviceWithZIO[Svc](_.method(a))` directly at call sites |

## Decision Guide: Which Abstraction?
> Deep dive: references/when-to-use-what.md

| Need | Use | Not |
|---|---|---|
| Service wiring + lifecycle | `ZLayer` | Manual instance passing |
| Single shared mutable state | `Ref` | `var`, `AtomicReference` |
| Multi-variable atomic updates | `STM` / `TRef` | Multiple `Ref` updates |
| Work distribution (one consumer per item) | `Queue` | `Hub` |
| Broadcast (all consumers get every item) | `Hub` | `Queue` |
| Retry with backoff / repeat on schedule | `Schedule` | Manual recursion |
| One-shot synchronization between fibers | `Promise` | `Queue`, `Ref` |
| Limit concurrent access | `Semaphore` | Manual counting |
| Incremental / infinite data processing | `ZStream` | `ZIO` returning `List` |
| Resource with finalization | `Scope` + `acquireRelease` | `ensuring` |
| Low-level fiber control | `Fiber` + `fork` | Default; prefer `foreachPar`/`zipPar` |

## Key Patterns
> More patterns: references/minimal-working-patterns.md | references/idioms.md

### Service Pattern (Repository variant)
```scala
trait UserRepo { def findById(id: UUID): IO[RepoError, Option[User]] }

case class PostgresUserRepo(ds: DataSource) extends UserRepo {
  def findById(id: UUID): IO[RepoError, Option[User]] =
    ZIO.attemptBlocking(query(ds, id)).mapError(DbError(_))
}

object UserRepo {
  val layer: ZLayer[DataSource, Nothing, UserRepo] = ZLayer.derive[PostgresUserRepo]
}
```

### Background Worker
```scala
def startWorker(queue: Queue[Job]): ZIO[Any, Nothing, Fiber.Runtime[Nothing, Nothing]] =
  queue.take.flatMap(job => process(job).ignoreLogged).forever.forkDaemon
```

### Retry with Backoff
```scala
val policy = Schedule.exponential(1.second) && Schedule.recurs(5)
ZIO.attempt(unstableCall()).retry(policy)
```

### Parallel with Limit
```scala
ZIO.foreachPar(urls)(fetch).withParallelism(10)
```

### HTTP Route (ZIO HTTP)
```scala
val route: Route[UserRepo, Nothing] =
  Method.GET / "users" / uuid("id") -> handler { (id: UUID, req: Request) =>
    ZIO.serviceWithZIO[UserRepo](_.findById(id)).map {
      case Some(user) => Response.json(user.toJson)
      case None       => Response.notFound
    }.orDie
  }
```
