## Index
[Sql vs NoSql](https://github.com/Danny7226/LearningSummary/blob/main/softwareengineering/README.md#sql-vs-nosql)

[Hash Ring](https://github.com/Danny7226/LearningSummary/blob/main/softwareengineering/README.md#hash-ring)

[B+ tree](https://github.com/Danny7226/LearningSummary/blob/main/softwareengineering/README.md#b-tree)

[Transaction and lock](https://github.com/Danny7226/LearningSummary/blob/main/softwareengineering/README.md#transaction-and-lock)

[Fallback vs Circuit Breaker](https://github.com/Danny7226/LearningSummary/blob/main/softwareengineering/README.md#fallback-vs-circuit-breaker)

[Multiple dispatch](https://github.com/Danny7226/LearningSummary/blob/main/softwareengineering/README.md#multiple-dispatch)

[A/B testing dial up](https://github.com/Danny7226/LearningSummary/blob/main/softwareengineering/README.md#ab-testing-dial-up)

[HashMap](https://github.com/Danny7226/LearningSummary/blob/main/softwareengineering/README.md#hashmap)

[Double-checked lock](https://github.com/Danny7226/LearningSummary/blob/main/softwareengineering/README.md#double-checked-lock)

[Continuous integration continuous deployment](https://github.com/Danny7226/LearningSummary/blob/main/softwareengineering/README.md#cicd-continuous-integration-continuous-deployment)

[Linux cmd](https://github.com/Danny7226/LearningSummary/blob/main/softwareengineering/README.md#linux-cmd)

[Throttling, token-bucket algorithm](https://github.com/Danny7226/LearningSummary/blob/main/softwareengineering/README.md#throttling-token-bucket-algorithm)

[RFC 7231 HTTP 1.1](https://github.com/Danny7226/LearningSummary/blob/main/softwareengineering/README.md#rfc-7231-hypertext-transfer-protocol-http-11)

[Rest API](https://github.com/Danny7226/LearningSummary/blob/main/softwareengineering/README.md#rest-api)

[Unicode](https://github.com/Danny7226/LearningSummary/blob/main/softwareengineering/README.md#unicode)

[Batch processing and stream processing]()

[Service discovery vs load balancer](https://github.com/Danny7226/LearningSummary/blob/main/softwareengineering/README.md#service-discovery-vs-load-balancer)

[Distributed Web Application quick roll-out](https://github.com/Danny7226/LearningSummary/blob/main/softwareengineering/README.md#distributed-web-application-quick-roll-out)

[Redis cache & hot key](https://github.com/Danny7226/LearningSummary/blob/main/softwareengineering/README.md#redis-cache--hot-key)

[Dynamo && how it handles hot partitions](https://github.com/Danny7226/LearningSummary/blob/main/softwareengineering/README.md#dynamo--how-it-handles-hot-partitions)

[Reversed index](https://github.com/Danny7226/LearningSummary/blob/main/softwareengineering/README.md#reversed-index)

[Version and Release Management](https://github.com/Danny7226/LearningSummary/blob/main/softwareengineering/README.md#version-and-release-management)

[Concurrent Write](https://github.com/Danny7226/LearningSummary/tree/main/softwareengineering#concurrent-write)

[Metrics](https://github.com/Danny7226/LearningSummary/tree/main/softwareengineering#metrics)

[NGINX](https://github.com/Danny7226/LearningSummary/tree/main/softwareengineering#nginx)

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
  * DynamoDB optimistic lock CAS is not thread safe, might still result in data loss, when write volume is huge
  * Optimistic is a mechanism that provides conditional check, but doesn't guarantee thread safe 
* Pessimistic locking, locks are placed automatically when data are getting processed
    * DynamoDB's transaction is based on pessimistic locking, instead of lock the rows, it monitors and rolls back transaction when another modification happens
      * So it's not a technically strict pessimistic lock, so write performance is still not perfect
      * Could use external lock system to synchronize concurrent requests to improve write performance
      * DDB transaction is to guarantee atomicity with rollback, but ConditionConflict might still happen (not thread-safe)
    * Optimistic locking is preferred for write-heavy (on different row) situation to avoid un-necessary lock gaining and release
    * Pessimistic locking is preferred for write-heavy (on same row) situation to provide CAS atomicity/isolation and avoid constant re-try 
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
* `tail -f`

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

### Rest API
* Batch API
  * Takes in multiple input object and respond with 207 multi-state when succeeded
  * Only TCP handshake once and make one connection, saves overhead when counting the overall latency
  * socket write takes around 5-10 micro-second, message write takes around 1 micro-second 
    * overhead is considered of a huge ratio if message size is small
* Get vs POST
  * Get is supposed to be idempotent, therefore will be potentially cached
  * Get is not intended to modify resources, when uploading parameter in body instead of querystring, it might result in compatibility issue
    * In smithy, @httpLabel -> url path, @httpQuery -> url query parameter, @httpPayload -> payload body 
  * Both querystring and request body are encrypted

### Unicode
* UTF-8, 8 bits (`255` in decimal, `ff` in hex) to encode common European characters, but takes up to 32 bits to encode some
  * 1 - 4 bytes
  * UTF-8 is compatible with ASCII. When ASCII is mostly used characters, storage using UTF-8 requires only 1 byte
  * UTF-8 file contains only ASCII characters that have the same encoding as ASCII file
* UTF-16, 
  * 2 or 4 bytes
* UTF-32
  * 4 bytes

### Batch processing and stream processing
* Stream processing put events in a stream, events can be partitioned to improve scalability and replicated to improve partition tolerant
* Batch processing are usually using Batch technique (apache hadoop) and map reduce to segment data into batches and process one by one
* Hybrid - Map reduce data into chunks and store in Object storage such as S3. Use queue message system and a bunch of works to process these data
    * Faster than Batch process and slower than stream processing

### Service discovery vs load balancer
* Service discovery acts like a facade to provide endpoints of services to make endpoint changes easier
* Service discovery provides ability to provide client cache
* load balancer acts as dispatcher to dispatch requests to servers to make node addition/deletion easier

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

### Reversed index
* Reversed index or inverted index is contrast to forwarded index
* instead of indexing the `key` in the data model, index `value` or keywords
* Better for full context search
* 
### Version and Release Management


### Websocket vs HTTP
* Websocket complements the long-lasting communication in web world, which is the missing part of HTTP
* Single handshake is enough for establishing long communicating session for websocket, whereas TCP handshake is required for each individual HTTP connection
* Starting HTTP2.0, mutiplexing enables multiple requests within same connection, HTTP3.0 provides multiplexing with multiple streams to mitigate head-of-line-blocking, but still not long-lasting enough
* Websocket is bidirectional whereas http is single-direction
* Usually the HTTP request is initiated by client, server cannot notify client on its own
* Websocket has lighter weight header so that communication expense is low than HTTP
* Websocket is initiated with an HTTP request with keywords in heads, like `Upgrade: websocket` and `Sec-WebSocket-Key` to request upgrading protocol from HTTP to Websocket
  * If server supports and agrees, http 101 status code will be returned (1xx informational, request received, continue processing)
  * 101 response with headers `Connection: Upgrade` and `Sec-WebSocket-Accept: xxx`
  * Heartbeat mechanism is used to maintain connections of websocket between client and server

### TCP vs UDP and IP
* TCP is connection oriented, which guarantees message delivery, in-order deliver, no package loss. It tracks the status of each connection
* IP, internet protocol, defines the format of the network layer packets to enable transmission navigation to make sure they can be delivered to the right address
* IPv4 is decimal notation `123:89:46:72` denotes `01111011:01011001:xxxxxxxx:xxxxxxxx` 4 bytes, 32 bits -> 4.2 billion addresses 4.2*10^9
* Ipv4 is hex notation 16 bytes, 128 bits
* NAT（Network Address Translation) was used to translate IP address in different networks
  * It could be used to translate private IP address to public address, which mitigate IPv4 shortage also hides the topological details of inner net. IPv6 makes it totally optional
* IP is the virtual location on the Internet, whereas Mac is the physical address/identifier of each device that can be connected to the Internet

### Concurrent write
* When write traffic volume is huge and concurrent writes happen on the same resources (say, to update a list field in a json document)
  * The request is to add an item in the list, or simply overwrite the whole list
  * Add an item in the list can provide higher success rate (availability) by using async process to serialize requests, but higher latency
  * Simple overwrite provides immediate response to clients, but need to introduce a mechanism to avoid data loss (CAS, or pessimistic lock for all update operations to ensure stronger consistency than CAS)
  * Or simply think back from the beginning, is it possible to normalize the data model so that this list field in document can be normalized into multiple entries to avoid concurrent update on shared resource
* When write traffic volume is not huge, simple optimistic lock with database's build-in CAS (usually not thread safe) is sufficient for most cases
  * Variant table (ability to get paginated results of an item's all variants)
  * `pk: {parentItemId}, sort: variant_timestamp_{variantItemId}, ...columnsOfOtherAttributes`

### Metrics
* How to use a single metrics indicate the overall availability
  * Available call emit data point 1, unavailable call emits datapoint 0
  * Need to make sure the emitted datapoint is binary so that the sample count represents the total call count
    * Average: availability (average is the sum/sample count)
    * Sum: successful request count
    * Sample count: total request count
    * Sample count - Sum: failed request count

### NGINX
* https://nginx.org/en/
* NGINX instance can be used as proxy server as well as to serve static contents
* Example of nginx config file
  * Longest location that matches with the input URI will be selected
  * `root` outside of `location` will be used as default in location blocks without a `root` directive
  * In example below, `/images/good.png` will fetch the assets located at `/data/images/good.png`
  * `/someother/good.png` will fetch the assets located at `/data/up1/someother/good.png`
  * location also supports regex. NGINX will remember the longest matching location to use if no regex matches
```agsl
http {
  server {
      listen 8080;
      root /data/up1;
  
      location / {
      }
  }

  server {
      location / {
          proxy_pass http://localhost:8080;
      }
  
      location /images/ {
          root /data;
      }
  }
}
```

* Load balancing
  * HTTP Load balancing
```agsl
http {
    upstream backend {
        ip_hash; # if not specified, round robin by default, other algo such as `hash $request_uri consistent`
        server backend1.example.com;
        server backend2.example.com down; # marked down temporarily to preserve the current hashing of client IP addresse 
        server 192.0.0.1 backup;
    }
    
    server {
        location / {
            proxy_pass http://backend;
        }
    }
}

```
