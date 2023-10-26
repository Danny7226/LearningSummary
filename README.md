# LearningSummary

Science and engineering are means of spiritual development. Precisely identifies the problem allows me to precisely recognize this world

## Index
[Sql vs NoSql](https://github.com/Danny7226/LearningSummary#sql-vs-nosql)

[Hash Ring](https://github.com/Danny7226/LearningSummary#hash-ring)

[B+ tree](https://github.com/Danny7226/LearningSummary#b-tree)

[Spring MVC](https://github.com/Danny7226/LearningSummary#spring-mvc)

[Service discovery vs load balancer](https://github.com/Danny7226/LearningSummary#service-discovery-vs-load-balancer)

[Transaction and lock](https://github.com/Danny7226/LearningSummary#transaction-and-lock)

[Java memory model](https://github.com/Danny7226/LearningSummary#java-memory-model)

[Weaving](https://github.com/Danny7226/LearningSummary#weaving)

[Dynamic proxy JDK](https://github.com/Danny7226/LearningSummary#dynamic-proxy-jdk)

[Tomcat, java EE, java servlet, and socket](https://github.com/Danny7226/LearningSummary#tomcat-java-ee-java-servlet-and-socket)

[Fallback vs Circuit Breaker](https://github.com/Danny7226/LearningSummary/tree/main#fallback-vs-circuit-breaker)

[Multiple dispatch](https://github.com/Danny7226/LearningSummary#multiple-dispatch)

[A/B testing dial up](https://github.com/Danny7226/LearningSummary#ab-testing-dial-up)

[HashMap](https://github.com/Danny7226/LearningSummary#hashmap)

[Distributed Web Application quick roll-out](https://github.com/Danny7226/LearningSummary#distributed-web-application-quick-roll-out)

[Double-checked lock]()

## Topics
### Sql vs NoSql
Structural query language (SQL) is a domain specific Lange(DSL) designed for relational database manage system (RDBMS)

Relational database bases on relational model of data, and was proposed by E.F.Codd in 1970. It was invented back than and fit well with the data access pattern.

Storage efficient was important as the cost was high (however, storage cost lowers as Moore's law), and online analysis processing was favoring a query engine that allows performer to look up data from any angles

Couple of comparisons

* Relational db provides structural data, and therefore reduces data duplication
* There are 5 level of normal forms, which help tp normalize data model in relational database
* Relational db allows data lookup from any angel, whereas NoSql lookup requires a pre-plan out for the data model based on access pattern
* NoSql db access data with a partition key, which tells where to find the data quickly therefore can be scaled up horizontally
* Partition key helps locate the physical data center. With a B+ tree, compute complexity is O(logn)
* Relational database is not easy to scale as JOIN operation is time expensive as size goes larger and larger

### Hash Ring
* Hash ring favors consistent hashing, meaning less re-shuffled when it comes to node deletion/addition
* K-position scatting is used to avoid uneven distribution of nodes. Results in k times re-shuffles when adding/deleting a new node
* Replication is used to improve availability. When hash looking up, next node becomes next N nodes.
* Eventual consistency provides faster read/write. For highest throughput, set W and R to be 1, meaning as long as one node performs operation successfully, return

### B+ tree
* m-way search tree has at most m children with m-1 keys
* m-way ST becomes B tree when a root has at least 2 children and other nodes has at least m/2 children, in the meantime, tree would be constructed bottom up (making sure all leaf at the same level)
* B tree becomes B+ tree when each node has its presents in the leaf, leaf are linked and only leaf has pointers to the record (disk block).
* B+ tree is used to optimize disk-based storage indexing. (A data-structure represents multi-level indexing )
* B+ tree has leaf nodes linked for faster range operations
* Direct lookup takes O(logn) compute complexity

### Spring MVC
* Spring MVC takes in http request in DispatchServlet (single entry point)
* DispatchServlet has WebApplicationContext spawned when server spins up
* DispatchServlet will dispatch requests to individual controller
* DispatchServlet is essentially a java servlet, which opens a java socket bonded to a certain port
* DispatchServlet acts as proxy so that it loads requests to RestController in different url path
* Root context is used by controllers, whereas WebApplicationContext is used by front controller (DispatchServlet)
* RestController returns data in textual format, whereas Controller returns the data to a ViewResolver

### Service discovery vs load balancer
* Service discovery acts like a facade to provide endpoints of services to make endpoint changes easier
* Service discovery provides ability to provide client cache
* load balancer acts as dispatcher to dispatch requests to servers to make node addition/deletion easier

### Transaction and locking
* Optimistic locking, locks are placed automatically when data is about to save
* In DynamoDB, optimistic lock is achieved by version attribute to detect whether a data is modified prior to updating it
* Pessimistic locking, locks are placed automatically when data are getting processed
  * DynamoDB's transaction is based on pessimistic locking
  * Optimistic locking is preferred for read-heavy situation to avoid race, deadlock, and un-necessary lock gaining and release
  * Pessimistic locking is preferred for write-heavy situation to avoid constant re-try
* In order to help understand Pessimistic lock, one example is JPA
  * JPA provides PESSIMISTIC_READ to maintain a shard lock, where data can be read when someone helds a shard lock
  * PESSIMISTIC_WRITE provides an exclusive lock where data cannot be READ, WRITE, UPDATE
  * PESSIMISTIC_FORCE_INCREMENT provides a similar exclusive lock, where data version entity will be incremented afterwards
* Explicit Locking, locks are placed manually
* Reference https://www.baeldung.com/jpa-pessimistic-locking

### Java memory model 
* https://medium.com/platform-engineer/understanding-java-memory-model-1d0863f6d973
* Java source code will be compiled by javac(java compiler) into .class files
* .class files are data in bytecode format in binary. People might think it's hexadecimal, but it's actually not. Hexadecimal representation is just a way of viewing binary data
* Java Runtime Environment (JRE) will take .class files. Classloader loads classes into JVM in runtime
* Classloader doesn't change the format of data, it simply loads files into memory
* Bytecode is platform independent, meaning it can be executed in any compatible JVM platforms. (Windows, Linux, OSX have different implementations of JVM, JVM acts as an abstract layer between bytecode and hardware OS)
* We all know static fields are stored into JVM heap instead of stack (each thread has its own stack) memory
* Even within threads, thread creates local objects, which are only visible to themselves. Note that these local objects also stored in heap instead of stack
* Thread maintains a reference in stack pointing to the actual object in heap
* Java garbage collection scans heap memory and cleans objects that are no longer referenced
* Static fields points to the same reference in heap, and therefore multiple threads accessing same static class/method/fields might cause race condition. However there is no guarantee of the visibility for all threads to see the changes
* java volatile is used to make sure data is written into main memory and visibility to all threads

### Weaving
* Weaving is a technical to manipulate byte-code java classes in JVM 
* Compile time weaving happens when javac compiles .java into .class
* Post-compile weaving happens to .class files
* Load-time weaving in AOP happens when java class loader loads classes files into JVM
* Runtime weaving happens in JVM (after class loader has loaded .classes files into JVM). (***Quesiton, relation to java agent?***) 
* Spring AOP uses a proxy based runtime weaving, and it uses java reflection fundamentally
* Compile time weaving cannot defer decision in runtime, but is better to debug to know problems fast
* Post-compile weaving works well with 3rd party code, which developers don't have source code of in general
* Load-time weaving introduce extra overhead when JVM/application starts up, or server spins up (as class loader needs to spend extra time to weave when loading classes into JVM)
* However, LTW provides feasibility to decide if/what to weave, and bytecode can be manipulated without change source code
* Load-time weaving keeps source code free of aspect related code (so that LTW are used at certain time not all time, say performance monitoring and debugging runtime) https://github.com/indrabasak/spring-loadtime-weaving-example
* LTW fine-grained control on when and where to apply aspects
* Usually config files are load-time-weaving enabled (***needs to do more research***)

### Dynamic proxy JDK vs CGLib
* Proxy patten use case is for access control, such as auth, throttle, delegation, and not about changing behaviors
* Dynamic proxies differ from static proxies in a way that they do not exist at compile time. Instead, they are generated at runtime by the JDK and then made available to the users at runtime
* Spring AOP weaving use JDK dynamic proxy by default. It creates a proxy implements the interface of the target objects and delegate method calls (this requires target object implements an interface)
* CGlib creates proxy by subclassing (extends). Because of this, class or methods should not be final


### Tomcat, java EE, java servlet, and socket
* Apache tomcat is a web server and java servlet container (servlet engine)
* Apache tomcat is not directly built upon java ee, but java se (standard edition)
* However, most application build web application by deploying WAR (web application archive) onto tomcat servlet engine
* Servlet is simply a java class that deals with http request and gives http response
* Java servlet is not directly communicating with java socket. Instead, tomcat (servlet enginer) directly uses java socket listening to a port to establish network transmission
* Tomcat (servlet contain) manages java servlet lifecycles, dispatch requests into appropriate servlet. Servelet can generate HTML, execute logic, access DB and etc
* Java socket fits into transport layer in OSI model (layer 4), socket is a software object representing an IP + port number
* Port is a logical entity. Port, along with IP, uniquely identifies the destination (endpoint) for a data packets in network

### Fallback vs Circuit Breaker
* Fallback provides alternative behavior for a graceful degradation
* Circuit breaker is a mechanism prevents resource exhaustion from repetitive calls to a failing component by introducing CLOSED, OPEN, HALF_OPEN states
* Circuit breaker and fallback are usually used together
* Circuit breaker can be often used in the ServiceClientConfig, concept are but not limited to timeout, retry

### Multiple dispatch
* Multiple dispatch is a pattern of some object-oriented programming languages, where the resolution of a method call depends on types of multiple Objects
* Visitor pattern utilizes double dispatch
* Collision in single-dispatch fashion
  ```agsl
  class Bus extends Car implements Collapsable {
    collapseWith(Collapsable against) {
        if (against instanceOf Platform)
          this.destroy();
          against.survive();
        } else if (against InstanceOf Car) {
          this.destroy();
          against.destroy();
        }   
  }
    
  // this is the application code
  bus.collapseWith(subway);
  ```
* For scenarios where resolution of a method depends on the characteristics of 2 objects, double-dispatch fashion provides a cleaner and better-organized program
   ```agsl
    class Bus extends Car implements Collapsable {
      collapseWith(Collapsable against) {
        this.destroy();
        against.collapseWith(this);
    }
  
    class Subway implements Platform {
        collapseWith(Car car) {
            this.survive();
            // ...log car and etc.
        }
  
        collapseWith(Platform platform) {
            this.destroy();
            // ...log platform and etc.
        }
    }
    
    // this is the application code
    bus.collapseWith(subway);
    subway.collapseWith(subway)
    ```
  
### A/B testing dial up
* Client-Id will be hashed to provide uniform distribution and then mapped to a range to determine which treatment to apply
* Client-Id will always to mapped to the same bucket in the range as long as hashing and mapping algorithm doesn't change, which shouldn't
* Pre-analysis exposure might happen during gradually dial-up
* Clean dial up
  * When dialing-up gradually, the mixed treatment clients are often hard to analysis cuz they maybe become ineligible for the pre-exposure
    * Say control/treatment ratio dial up from 9:1 to 5:5. Certain clients will be treated in control first, but then fall into treatment category
  * This mixed treatment might not be a noise when dialing-up is purely to test the software functional integrity
  * But when it comes to business analysis, mixed treatment often cause a great noise
  * Instead of dialing up treatment percentage, we could use exposure control, which is essentially another A/B testing gate in front of our main A/B testing
  * What it does is, it will choose certain percentage of customers (say 10 percent) in the experiment, and then apply the control/treatment ratio to this 10% customers
    * This makes sure a certain customer will never be treated differently during dial-up
    * We can gradually dial up the exposure percentage (from 10 - 100%), instead of control/treatment ratio
    * And then, we only analyze people that fall in the gating treatment scope. And each individual client will be treated consistently

### HashMap
* HashMap uses array and linkedlist fundamentally to implement
* When the size of a linkedlist goes greater than 8, HashMap will expand array size up to 64 first
* If size of the HashMap array is greater than 64 (2^6), and size of linkedlist is greater than 8 (2^3), HashMap will treeify LinkedList into red/black tree
* Red/Black tree is optimized than AVL (balanced-binary search tree) in the sense of faster insertion and removal operations (but takes more memory as a tradeoff)
* AVL is a good data-structure in memory, whereas B tree (m-way search tree) is good for disk data storage
* Define exp(x,y) means x to the power of y
  * Size of HashMap is exp(2,x), instead of an arbitrary integer. The reason for this is there is an `(n-1) & hash` operation to determine which index/bucket to put data in the array.
  * if size is in the form of exp(2,x), mod operation is fast as it can be calculated with bit operation

### Distributed Web Application quick roll-out
https://www.linkedin.com/feed/update/urn:li:activity:7123372072059248640/
1. Web-Based Security Firewall: To safeguard against common web threats like XSS and SQL injection, as well as to manage throttling and domain-specific traffic control.
2. Content Delivery Network (CDN): Serving as an edge caching and redirection solution, it can vend static assets for your application's frontend, cache responses, and define redirect behaviours.
3. DNS Routing: Utilized to route domain names to actual IP addresses for further request processing.
4. Mediator Gateway: Acting as a mediator layer to reduce chaotic dependency relationships, often integrated with an aspect layer of authentication and authorization, delegated to third-party providers.
5. Load Balancing: For distributing traffic among distributed web servers, with delicate routing principles for even traffic distribution and geographic-based routing.
6. Docker management: for orchestrating docker images, facilitating horizontal scaling, rollback, and recovery. Also provides an extra layer of abstraction for smooth migration towards other platforms.
7. Database: Even you don’t own the critical data you need, a database is still necessary to persist top-level entity configurations through some control plane APIs
8. Encryption Management: Emphasizing client-side and server-side encryption for secure data protection.
9. Caching: Employed to reduce repetitive IO load on the database by storing frequently accessed data.
10. Asynchronous Data Processing: Depending on your business needs, asynchronous or event-driven data processing mechanisms can be vital. Queueing service providers ensure strong message delivery consistency and robust failure handling and recovery. Streaming services are preferred for high throughput, with a focus on shard-level processing.
11. Proxy to internal data/Data Plane APIs: To tunnel into internal dependencies, who have the data you need, in order to support business experience.

### Double-checked lock
* To reduce lock overload and achieve lazy initialization
* First if check is to prevent constantly gaining and releasing lock
* Second if check is to make sure actually un-assignment after gaining lock
* `synchronized` is to make sure `allocate memory` `construct object` `assign object to memory` are atomic
  * if not atomic, memory might be allocated but object not assigned
  * another thread might return memory with unassigned object, which might result in error
* `volatile` is necessary as instance state needs to be visible to all threads
  * If initialization made by thread A is not visible, thread B might override initialization.
```agsl
Class LazyInitialization {
    private volatile static LazyInitialization instance;
    public static getInstance() {
        if (instance == null) {
            synchronized(LazyInitialization.class) {
                if (instance == null) {
                    instance = new LazyInitialization();
                }
            }
        }
    }
}
```