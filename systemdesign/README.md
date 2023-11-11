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
    * what is the expectation of write-to-read delay
      * If low delay, need to think async
      * If high delay is permitted, batch or stream processing is allowed
    * Latency
      * low latency required, means data needs to be aggregated
      * high latency permitted, means we could use query language (such as MySql) to query relational tables
  * What is the cost expectation of the system
    * value development cost, use open-source
    * value maintenance more, use cloud service
* Put down functional and non-functional requirements accordingly
  * functional requirements are the APIs we are going to have
    * getViewCount(videoId)
      * getCount(videoId, eventType)
        * getStat(videoId, eventType, func) // func: count, sum, avg
          * it goes even further and more abstract depends on what is considered as the top level entity of our system
      * the video is considered as the input entity, this is negotiable when we get to the domain data model to see what is the best fit here
  * non-functional requirements are ones like fast, fault tolerant, SLA, secure
    * Scale (millions of TPS)
    * Performance (low latency, tens of milliseconds)
    * Availability (no single point of failure, survives hardware/network failure)
  * Define domain model and data model (ticket, parking lot, parking slot) (video, view...)
    * What data we have and can potentially store
      * individual event (videoId, viewId, timestamp, viewerIdentity, etc.)
        * fast write, can slice and dice data however we want, can recalculate data and do analysis
        * Slow read, high storage cost for large scale of data
      * aggregated data of events (videoId, timeframe, total view)
        * Fast read of our own service, data is ready to use for other decision-making services
        * can only query the way how data is aggregated, requires building data aggregation pipeline, hard for bug recovery as we don't have raw data on premise
    * How to store data
      * Ask for expected delay
        * within minutes, stream processing - process data on the fly and store data for few days to a week
        * within hours, batch processing - store events in data lake and process them in background

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
