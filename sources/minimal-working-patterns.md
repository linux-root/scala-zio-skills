### **1. Service Pattern with ZLayer**
*   **Explanation**: This pattern **decouples a service's contract (interface) from its implementation**, following "Onion Architecture" principles. It uses a **trait** to define capabilities, a **concrete class** to implement them (receiving dependencies via the constructor), and a **ZLayer** in the companion object to handle initialization and finalization.
*   **Code Example**:
    ```scala
    // 1. Interface
    trait Greeter { def sayHello(name: String): UIO[Unit] }
    
    // 2. Implementation with constructor dependencies
    case class GreeterLive(console: Console) extends Greeter {
      def sayHello(name: String): UIO[Unit] = console.printLine(s"Hello, $name!").orDie
    }
    
    // 3. Layer for dependency injection
    object Greeter {
      val layer: URLayer[Console, Greeter] = ZLayer.derive[GreeterLive]
      def sayHello(name: String): URIO[Greeter, Unit] = ZIO.serviceWithZIO(_.sayHello(name))
    }
    ```

### **2. Repository Pattern**
*   **Explanation**: A specialized version of the Service Pattern used to **encapsulate data access logic** (e.g., SQL queries or API calls). It allows the rest of the application to interact with data using domain models while **hiding storage-specific details** like database connections or transaction management.
*   **Code Example**:
    ```scala
    trait UserRepo { def findById(id: UUID): Task[Option[User]] }
    
    case class PostgresUserRepo(db: Database) extends UserRepo {
      def findById(id: UUID): Task[Option[User]] = 
        ZIO.attemptBlocking(db.query(s"SELECT * FROM users WHERE id = $id")) // simplified
    }
    
    object UserRepo {
      val layer: ZLayer[Database, Nothing, UserRepo] = ZLayer.derive[PostgresUserRepo]
    }
    ```

### **3. HTTP Handler Pattern**
*   **Explanation**: In ZIO HTTP, transactions are modeled as **pure functions from a Request to a ZIO effect producing a Response**. Routes are defined using a **declarative DSL** (RoutePattern) that enables type-safe extraction of path parameters and query strings.
*   **Code Example**:
    ```scala
    import zio.http._
    
    val userRoute: Route[UserRepo, Nothing] = 
      Method.GET / "users" / uuid("id") -> handler { (id: UUID, req: Request) =>
        UserRepo.findById(id).map {
          case Some(user) => Response.json(user.toJson)
          case None       => Response.notFound
        }.orDie
      }
    ```

### **4. Background Worker / Job Processing**
*   **Explanation**: This pattern uses **Queue** for work distribution and **forkDaemon** to run long-lived "worker" fibers that outlive the scope of their parent. It allows the main application to submit tasks to a buffer while **independent fibers process them concurrently** in the background.
*   **Code Example**:
    ```scala
    def startWorker(queue: Queue[Job]): ZIO[Any, Nothing, Fiber.Runtime[Nothing, Nothing]] =
      queue.take
        .flatMap(job => process(job).ignoreLogged) // process job
        .forever
        .forkDaemon // runs in global scope
    ```

### **5. Retry and Scheduling**
*   **Explanation**: ZIO uses the **Schedule** data type to describe **composable strategies** for retrying failed effects or repeating successful ones. These "blueprints" can combine logic for **delays (exponential backoff), repetition limits, and conditions**.
*   **Code Example**:
    ```scala
    val retryPolicy = 
      Schedule.exponential(1.second) && Schedule.recurs(5) // backoff + max retries
      
    val robustAction = 
      ZIO.attempt(unstableCall()).retry(retryPolicy)
    ```

### **6. Parallel Processing**
*   **Explanation**: High-level operators like **foreachPar** and **zipPar** allow for executing multiple effects concurrently without the overhead of manual fiber management. These operators follow **structured concurrency**, ensuring that if one sub-task fails, all siblings are automatically interrupted to prevent resource leaks.
*   **Code Example**:
    ```scala
    val urls = List("url1", "url2", "url3")
    
    // Process all URLs in parallel with a concurrency limit of 5
    val results = ZIO.foreachParN(5)(urls) { url =>
      HttpClient.get(url)
    }
    ```
