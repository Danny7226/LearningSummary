# LearningSummary

Science and engineering are means of spiritual development. Precisely identifying the problem allows me to precisely recognize this world

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

[Double-checked lock](https://github.com/Danny7226/LearningSummary#double-checked-lock)

[Continuous integration continuous deployment](https://github.com/Danny7226/LearningSummary#cicd-continuous-integration-continuous-deployment)

[Redis cache & hot key](https://github.com/Danny7226/LearningSummary#redis-cache--hot-key)

[Linux cmd](https://github.com/Danny7226/LearningSummary/tree/main#linux-cmd)

[Throttling, token-bucket algorithm](https://github.com/Danny7226/LearningSummary/tree/main#throttling-token-bucket-algorithm)

[Version and Release Management](https://github.com/Danny7226/LearningSummary/tree/main#version-and-release-management)

[Dynamo && how it handles hot partitions]()

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
  * CAS (compare and swap) concept is used in optimistic locking
  * CAS is introduced to make sure a thread updating a shared memory would fail if the shared memory has been updated to another
  * CAS has ABA problem, meaning even comparedValue and expectedValue are the same, doesn't mean it has not been updated
  * VersionNumber/TimeStamp is often used along with CAS to achieve optimistic locking
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
* First `if` check is to prevent constantly gaining and releasing lock
* Second `if` check is to make sure actually un-assignment after gaining lock
* `synchronized` is to make sure `allocate memory` `construct object` `assign object to memory` are atomic
  * Reason for this is, `new Object()` is not atomic 
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
        } else {
            return instance;
        }
    }
}
```
* `synchronized void method() {}` lock an object instance
* `synchronized static void method() {}` locks the whole class (all instances of this class)
* `synchronized(this)` locks an object, `synchronized(A.class)` locks the whole class

### CICD (Continuous integration continuous deployment)
* feature dial up can be done by both CICD or weblab
* CICD provides automatic rollback through continuous deployment but requires all changes of feature roll-in one-box stage at once
  * Usually requires a separate git branch to store all features and resolve conflict at once
* weblab provides ability to merge commits to mainline to production while hiding feature code behind a feature flag
* CICD is more useful when during daily maintenance and small feature roll-out
* Weblab is more useful when doing big feature launch since weblab dial-up/down is faster than code deployment

### Redis cache & Hot key
* Use write read node to deal with different operations (replica)
* Use cluster of nodes to distribute hot keys into multiple nodes (re-partition data)
* Move hot key data into higher level cache, such as JVM memory
* Cache Penetration (invalid key request land on database)
  * Set up invalid key cache
  * Valid request and return in application layer (request field range, format and etc)
* Cache Hotspot Invalid (Hot key request all land on database as cache got invalid all of a sudden)
  * Pre-heat data into cache before hot situation
  * Set long TTL for hot key
  * Setup exclusive lock for hot key in application layer
* Cache avalanche (large amount of hot key get invalid in short amount of time)
  * Use clusters to avoid single point of failure
  * Set different ttl for hot keys so that hot key cache won't be invalid all at once
  * L2 cache in JVM memory

### Linux cmd
* `ls -l`: long list
* `ls -li`: long list with inode
* `ls -lih`: h means in size human readable manner
* `du -sh .`: disk usage, `s`: summary, `h`: human readable
* `df -h`: disk free, check disk info for current mounted file systems
* `inode`: is the index of each file, is unique within each file system
  * In linux, everything is considered as a file
* File types
  * -: regular file
  * d: directory
  * l: link file
  * s: socket file, for network
* `chmod 744 {filename.surfix}` = `chmod ugo+rwx` = `chmod a+rw` = `chmod +rwx`
  * `r`: read, `w`: write, `x`: execute
  * `u`: user, `g`: group, `o`: others, `a`: all
  * 7 in bits `111`
    * read: 4 `100`, write: 2 `010`, execute 1 `001`
    * 764 means, user: 7 (rwx), group 6 (rw), others 4 (read)
* File link
  * Hard link: points to the actual data, data will only be deleted when all inode deleted
    * Cannot cross file systems as inode would be the same for hard linked files, and each file system has their own inode management 
    * `ln {originalFile} {newFile}`
  * Soft link: creates a link points a file to another file
    * Can cross file systems
    * `ln -s`

### Throttling, token-bucket algorithm
* Each transaction/request takes one token away from the bucket
* max_capacity: the total maximum number of token in the bucket, defines the burst rate
* max_rate: the number of tokens to refill into the bucket when it's not full, it defines the throttling TPS

### Version and Release Management

### Dynamo && how it handles hot partitions
* Dynamo db has a structure of { partition router => partitions[1, 2, 3...] }
* By default, Dynamo enables 3000 RCU and 1000 WCU per sec per partition node
* At max, 10 GB per partition and 400kb per item, (25000 items per partition)
* The above provisioned capacity will be evenly distributed onto each partition
* If partition has items more than 10GB, dynamo moves half of the data into another partition under the hood and maintains a mapping in the router
* Provisioned capacity, in general, can be achieved by separating write/read nodes, allocate replicas, applying master-replica nodes architecture
* Instant adaptive capacity allows burst traffic not being throttled even exceeding partition throughput capacity
  * When certain items becomes popular, Dynamo does a thing called 'isolate frequently accessed items'
  * It basically re-balances the partitions so that one partition will be dedicatedly serving one particular hot item
  * Keep in mind, adaptive capacity cannot exceed table's provisioned capacity or partition's maximum capacity (3000 RCU and 1000 WCU)
  * Dynamo will not re-shuffle item collections across multiple partitions if there is an LSI (local secondary index)
* Reference [this](https://deeptimittalblogger.medium.com/dynamodb-data-partitioning-is-the-secret-sauce-c810eb9f80ca)
* Dynamo has LSI and GSI.
  * LSI and GSI are essentially copies of data from main table
  * LSI has to be specified during table creation and cannot be added to an existing table
    * This is because LSI are sort keys within partitions, and to maintain data in sorted order for easy adjacent access
  * GSI can be added any time as they are essentially partitions
* Dynamo supports 3 ways to modify its entities
  * Rest API
  * SDK for many program languages
  * CLI