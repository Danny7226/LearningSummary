## Index
[Sql vs NoSql](https://github.com/Danny7226/LearningSummary#sql-vs-nosql)

[Hash Ring](https://github.com/Danny7226/LearningSummary#hash-ring)

[B+ tree](https://github.com/Danny7226/LearningSummary#b-tree)

[Transaction and lock](https://github.com/Danny7226/LearningSummary#transaction-and-lock)

[Fallback vs Circuit Breaker](https://github.com/Danny7226/LearningSummary/tree/main#fallback-vs-circuit-breaker)

[Multiple dispatch](https://github.com/Danny7226/LearningSummary#multiple-dispatch)

[A/B testing dial up](https://github.com/Danny7226/LearningSummary#ab-testing-dial-up)

[HashMap](https://github.com/Danny7226/LearningSummary#hashmap)

[Double-checked lock](https://github.com/Danny7226/LearningSummary#double-checked-lock)

[Continuous integration continuous deployment](https://github.com/Danny7226/LearningSummary#cicd-continuous-integration-continuous-deployment)

[Linux cmd](https://github.com/Danny7226/LearningSummary/tree/main#linux-cmd)

[Throttling, token-bucket algorithm](https://github.com/Danny7226/LearningSummary/tree/main#throttling-token-bucket-algorithm)

[RFC 7231 HTTP 1.1](https://github.com/Danny7226/LearningSummary/tree/main#rfc-7231-hypertext-transfer-protocol-http-11)

[Version and Release Management](https://github.com/Danny7226/LearningSummary/tree/main#version-and-release-management)


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

### RFC 7231 Hypertext transfer protocol HTTP 1.1
* Cacheable method, methods that doesn't rely on authoritative response (source of truth)
    * GET, HEAD, POST
        * HEAD is faster than GET, is usually used to compare if the data has been changed since the last cached response
            * This can be achieved by comparing the fixed-length Hash value of data
            * Hash values are usually used in HTTP transmission to determine if the downloaded data has been modified or corrupted
            * It's usually included in the Header, as HEAD method will cut off the message body and only sent headers back

### Version and Release Management
