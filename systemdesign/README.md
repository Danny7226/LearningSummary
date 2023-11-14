## Index
[Area of focus](https://github.com/Danny7226/LearningSummary/blob/main/systemdesign/README.md#area-of-focus)

[Service discovery vs load balancer](https://github.com/Danny7226/LearningSummary/blob/main/systemdesign/README.md#service-discovery-vs-load-balancer)

[Distributed Web Application quick roll-out](https://github.com/Danny7226/LearningSummary/blob/main/systemdesign/README.md#distributed-web-application-quick-roll-out)

[Redis cache & hot key](https://github.com/Danny7226/LearningSummary/blob/main/systemdesign/README.md#redis-cache--hot-key)

[Dynamo && how it handles hot partitions](https://github.com/Danny7226/LearningSummary/blob/main/systemdesign/README.md#dynamo--how-it-handles-hot-partitions)

[Reversed index](https://github.com/Danny7226/LearningSummary/blob/main/systemdesign/README.md#reversed-index)

[Search service](https://github.com/Danny7226/LearningSummary/blob/main/systemdesign/README.md#search-service)

[Data processing pipeline](https://github.com/Danny7226/LearningSummary/blob/main/systemdesign/README.md#data-processing-pipeline)

## System Design
### Area of focus
* Ambiguity
  * Who is user/customer
    * What data system need
    * How data and system interact
  * What is the scale/traffic of the system
    * Read traffic per sec
    * Write traffic per sec
    * Any spikes
    * How much data each read/write
  * What is the performance of the system
    * what is the expectation of write-to-read delay (affects how we process our data)
      * If low delay, need to think sync operations / stream processing (with in minutes)
      * If high delay is permitted, batch is allowed (with in hours)
    * Latency (affects how to store our data / data-model)
      * low latency required, means data needs to be aggregated
      * high latency permitted, means we could use query language (such as MySql) to query relational tables
  * What is the cost expectation of the system
    * value development cost, use open-source
    * value maintenance more, use cloud service
* Put down functional and non-functional requirements accordingly
  * functional requirements are the APIs we are going to have
    * Write down verb and input/output, then make some iterations  
    * getViewCount(videoId)
      * getCount(videoId, eventType)
        * getStat(videoId, eventType, func) // func: count, sum, avg
          * it goes even further and more abstract depends on what is considered as the top level entity of our system
      * the video is considered as the input entity, this is negotiable when we get to the domain data model to see what is the best fit here
  * non-functional requirements are ones like fast, fault tolerant, SLA, secure (CAP)
    * Scale (millions of TPS)
    * Performance (low latency, tens of milliseconds)
    * Availability (no single point of failure, survives hardware/network failure)
    * Consistency
* Define domain model and data model (ticket, parking lot, parking slot) (video, view...)
  * What data we have and can potentially store
    * individual event (videoId, viewId, timestamp, viewerIdentity, etc.)
      * fast write, can slice and dice data however we want, can recalculate data and do analysis
      * Slow read, high storage cost for large scale of data
    * aggregated data of events (videoId, timeframe, total view)
      * Fast read of our own service, data is ready to use for other decision-making services
      * can only query the way how data is aggregated, requires building data aggregation pipeline, hard for bug recovery as we don't have raw data on premise
  * What database to use / where to store data
    * Things to consider
      * How to scale write, read
      * How to make read, write fast
      * How not to lose data in case of hardware/network failure
      * How to achieve strong consistency, what are the tradeoff
      * How to recover data in case of outage
      * How to ensure data security
      * How to make it extensible for data model change in the future
      * Where to run and how much (cloud vs on-premises)
    * Sql, strong consistency, easy transaction 
      
      ![](https://github.com/Danny7226/LearningSummary/blob/main/systemdesign/assets/Sql.png)
      * Geographically read replicas for availability and partition tolerant
      * Sharding partition for scalability
      * Shard proxy for performance
      * proxy cluster router to maintain health check, serves as mediator and request router
    * NoSql (architecture are examples, not all NoSql are the same)
      
      ![](https://github.com/Danny7226/LearningSummary/blob/main/systemdesign/assets/NoSql.png)
      * Data nodes are treated equal and check heartbreak with no more than 3 and propagate (gossip protocol)
        * Decentralized, resilient, scalable
      * Read replicas and geographical replicas in the hash ring (consistent hashing) to provide scalability and partition tolerant
      * Quorum writes/reads results in eventual consistency and provides performance
        * Chosen availability over consistency - prefer to show stale data over no data at all
      * Node takes in request will be chosen as coordinator node, and route requests and return with a quorum defined
        * When quorum writes/reads, requests are async, and returns with only defined number of nodes succeed writing 
        * Which node to take in requests can be chosen round-robin, or geographically close or other algorithms
    * What is data model
      * Sql: define nouns and use foreign key to relate to different tables
        * data is normalized so that update of data won't require update here and there
          * Video Table
          * Video view stat Table
      * NoSql: define access pattern
        * videoId, videoName, timestamp, viewCount
  * How to process and store data
    * How to process data to scale, reliable and fast
    * Ask for expected delay
      * within minutes, stream processing - process data on the fly and store data for few days to a week
      * within hours, batch processing - store events in data lake and process them in background
    * Aggregate data in memory before DB IO.
      * Read data for a certain amount of time before DB IO
    * Push/Pull: With the help of a queue/stream, you have event checkpoint and infra partitioning (for e.g. hash videoId)
      * Pull model is easy to scale by simply adding more workers 
      * Pull model is fault-tolerant, as data is always there and pull model can retry
      * Push model data would be lost if domain logic failed
* Components of service
  * High level first
    * How data get in and get out
    * ![](https://github.com/Danny7226/LearningSummary/blob/main/systemdesign/assets/full.png) ([Source](https://www.youtube.com/watch?v=bUHFg8CZFws&t=12s))
  * Partitioner Service Client
    * Blocking IO, one thread processing one socket connection. Thread local to look into states of each thread stack, easy to debug 
    * Non-blocking IO, requests are queued and processed by one thread. Result in higher throughput, but higher complexity of operations
    * APIGateway buffer and batch. Requests are sent into buffer and batched and sent to server together to increase throughput, lower cost. Also, higher complexity
    * Timeout
      * Connection timeout, how much client is willing to wait for a connection to establish (tens of milliseconds usually)
      * Request timeout, how much client is willing to wait for a request execution to complete
      * Retries, avoid single server failure. Also need to be careful with retry numbers to avoid retry storm
        * Exponential backoff and jitter
      * Circuit breaker to prevent endless retry with a failure ratio pre-defined and start to re-try after certain time
        * Introduce difficulty to test
  * Load balancer
    * Hardware millions of requests, Software load balancer
    * Network protocol
      * TCP lb, routes tcp connections without inspection content inside
      * Http lb, might look into the content and make routing decisions based on the content (cookie or header of the request)
    * Algorithms
      * Round-robin
      * least-connection
      * least-response time
      * Hash-based (request url, ip)
    * Load balancers' IP are provided to DNS, and server IPs need to be registered to Load Balancer
      * There are cloud technologies providing registration out of the box, e.g. Docker platform to register/unregister docker containers
      * Load balancer needs to do health check
      * Replica to provide high availability, secondary monitors primary and takes over (resides in different data center to prevent outage)
  * Partitioner service and partitions
    * Partition strategy
      * Hash (might result in hot-partition) with timestamp e.g. `hash(videoId)_1630`, `hash(videoId)_1700`
      * Split data into 2 partitions in hash ring
      * Reserve dedicated partition for hot data
    * Service discovery
      * Server side, allows client to simply call load-balancer, the discovery logic is hidden behind
      * Client side, allows client to call discovery service to get lists of registered endpoint
    * Partition replication to provide partition tolerant
      * Single leader replication, one leader maintained for the information of all partitions (followers)
        * Write/read talks to leader
        * If leader dies, a follower promotes to leader
      * Leader-less replication, partition nodes gossip to each other and any node can take in requests
      * Multi-leader replication, used for multiple data centers (different physical locations)
    * Message format
      * Textual: Json, xml, csv. Better for human readability and debug from dead letter queue
      * Binary: Thrift, Avro. Faster transmission and easier to parse for machine, as it's more compact
        * For e.g. Json requires fields included in the message, whereas binary message use fields tags, which is light in size when encoded, serves as alias to field names
        * Binary message format requires schema predefined and be aware of from both message producer and consumer sides.
  * Data storage for high volume data
    * Storage roll up
      * Fresh data will be stored in every minute granularity
      * After 2nd day, data will be stored every hour granularity
      * After a week, data will be stored every day granularity
      * After a month ...., we keep the latest in active DB (hot storage), rest in object storage (cold storage) such as AWS S3
  * Data query
    * Query result based on URL
    * Query aggregator to fetch recent data from hot storage, ancient data for cold storage
* OE
  * To identify bottlenecks and how to deal with them
    * Load testing (Apache JMeter)
    * Stress testing to a failure point
    * Soak testing to test high load over an extended period of time
  * Make sure service is running healthy
    * Monitoring and metrics with alarms
  * Make sure service produce accurate results
    * Weak testing, e2e testing
    * Strong testing, combine 2 processing systems' result and stitch them at the query system for comparison
    * 

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

### Search Service
* Proxy
    * Single entry to data center, provides load balancing, routing, monitoring, throttling, auth and various of benefits
* Ingestion
    * To avoid overwhelming load on data center
    * have queue/stream and workers to process at its own speed, ease data center load, provide failure recovery
* Back-fill
    * large data lake back-fill is huge burden to data center
    * Map reduce to break into smaller tasks and have workers working on them

### Data processing pipeline
* Ingesting
* Scoping
* Computing
