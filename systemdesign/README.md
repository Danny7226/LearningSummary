## Index
[Area of focus](https://github.com/Danny7226/LearningSummary/blob/main/systemdesign/README.md#area-of-focus)

[Search service](https://github.com/Danny7226/LearningSummary/blob/main/systemdesign/README.md#search-service)

[Data processing pipeline](https://github.com/Danny7226/LearningSummary/blob/main/systemdesign/README.md#data-processing-pipeline)

[System design case 1: design parking lot system](https://github.com/Danny7226/LearningSummary/tree/main/systemdesign#system-design-case-1-design-parking-lot-system)

[System design case 2: proximity service](https://github.com/Danny7226/LearningSummary/tree/main/systemdesign#system-design-case-2-proximity-service)

[System design case 2: Amazon phone tool service]()

[System design case 4: reviews with pagination](https://github.com/Danny7226/LearningSummary/tree/main/systemdesign#system-design-case-4-review-with-pagination)

[System design case 5: Monitoring JVM metrics](https://github.com/Danny7226/LearningSummary/tree/main/systemdesign#system-design-case-5-data-center-health)

## System Design
https://excalidraw.com/
### Area of focus
* Ambiguity (5 min)
  * Who is user/customer
    * What data system need
    * How data and system interact
  * functional requirements are the APIs we are going to have
    * Write down verb and input/output, then make some iterations
    * getViewCount(videoId)
      * getCount(videoId, eventType)
        * getStat(videoId, eventType, func) // func: count, sum, avg
          * it goes even further and more abstract depends on what is considered as the top level entity of our system
      * the video is considered as the input entity, this is negotiable when we get to the domain data model to see what is the best fit here
  * What is the scale/traffic of the system
    * Read traffic per sec
    * Write traffic per sec
    * Any spikes
    * How much data each read/write
  * What is the performance of the system (**availability/redundancy**, **consistency**, **partition-tolerance**)
    * what is the expectation of write-to-read delay (affects how we process our data)
      * If low delay, need to think sync operations / stream processing (with in minutes)
      * If high delay is permitted, batch is allowed (with in hours)
    * Latency (affects how to store our data / data-model)
      * low latency required, means data needs to be aggregated
      * high latency permitted, means we could use query language (such as MySql) to query relational tables
  * What is the cost expectation of the system
    * value development cost, use open-source
    * value maintenance more, use cloud service
* Define domain model and data model - Entity, ER, type and size of storage (10 - 15 min)
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
    * Sql, ACID transaction, analytics, data warehousing 
      
      ![](https://github.com/Danny7226/LearningSummary/blob/main/systemdesign/assets/Sql.png)
      * Geographically read replicas for availability and partition tolerant
      * Sharding partition for scalability
      * Shard proxy for performance
      * proxy cluster router to maintain health check, serves as mediator and request router
    * NoSql, easy read/write scaled (architecture are examples, not all NoSql are the same)
      
      ![](https://github.com/Danny7226/LearningSummary/blob/main/systemdesign/assets/NoSql.png)
      * Data nodes are treated equal and check heartbreak with no more than 3 and propagate (gossip protocol)
        * Decentralized, resilient, scalable
      * Read replicas and geographical replicas in the hash ring (consistent hashing) to provide scalability and partition tolerant
      * Quorum writes/reads results in eventual consistency and provides performance
        * Chosen availability over consistency - prefer to show stale data over no data at all
      * Node takes in request will be chosen as coordinator node, and route requests and return with a quorum defined
        * When quorum writes/reads, requests are async, and returns with only defined number of nodes succeed writing 
        * Which node to take in requests can be chosen round-robin, or geographically close or other algorithms
      * 3 types (column, document, key-value)
        * document db is wide-columned
    * What is data model
      * Sql: define nouns and use foreign key to relate to different tables
        * data is normalized so that update of data won't require update here and there
          * Video Table
          * Video view stat Table
      * NoSql: define access pattern
        * videoId, videoName, timestamp, viewCount
    * Calculate how much data storage is needed
      * DAU * daily write operation / user
* Components of service (30 - 45 min)
  * High level design (5 min)
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
* OE - operate-able, simple, extensible
  * To identify bottlenecks and how to deal with them
    * Load testing (Apache JMeter)
    * Stress testing to a failure point
    * Soak testing to test high load over an extended period of time
  * Make sure service is running healthy
    * Monitoring and metrics with alarms
  * Make sure service produce accurate results
    * Weak testing, e2e testing
    * Strong testing, combine 2 processing systems' result and stitch them at the query system for comparison

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

### System design case 1: design parking lot system
![](https://github.com/Danny7226/LearningSummary/blob/main/systemdesign/assets/parkinglot/textual.png)

* Ambiguity as mentioned in pic
* Functional non-functional features as mentioned in pic
* Domain/data model
  * What data we have
    * Data is not complicated here, we have the parking slot info when indexing, which might include the following
      * Parking building #
      * Parking slot #
      * Size (small, medium, large)
      * Occupied
    * When request/extend/pay-out a reservation, we have a reservation info, which might include the following
      * Reservation Id
      * Parking building #
      * Parking slot #
      * Status
      * Time spent (from, to)
      * Amount paid
  * Domain/data model as mentioned in pic
  * How to process
    * We could queue indexing requests for even-paced processing and failure recovery
      * Partition queues when index request is high (this is probably not needed)
      * Replica queue in different data center to provide failure-tolerant
    * We choose relational database for strong consistency when doing ticket and parking lot update transactions
    * Aggregate reservation requests within server runtime every second if the traffic volume is getting intense to avoid replicated DBIO connection overhead
    * Request reservation
      * Input (parking lot #, size, time needed)
      * Output (reservation #, parking slot #)
      * Scan parking lot # partition node to find right size
      * Update slot status to occupied
      * Put reservation entry
    * Extend reservation
      * Update end time in `reservation table`
    * Pay-out reservation
      * If within minutes, async, to even high volume traffic
        * Put message in queue and return success
      * If within a second, sync, use transactions
* System diagram
  * ![](https://github.com/Danny7226/LearningSummary/blob/main/systemdesign/assets/parkinglot/parkinglot.png)
  * Request reservation
    * Input (parking lot #, size, time needed)
    * Output (reservation #, parking slot #)
    * Scan parking lot # partition node to find right size
    * Update slot status to occupied
    * Put reservation entry
    * Extend reservation
      * Update end time in `reservation table`
    * Pay-out reservation
      * Sync way, use transactions
      * Async, put message in queue and return success

### System design case 2: Proximity Service
* Ambiguity
  * Customer
    * Business owner
    * User to query business locations given a location and search radius
  * Traffic volume
    * We have 50 million business
    * We have 100 million active users with 5 queries per day
  * Performance
    * write-to-read delay
      * It can take up to a day to update the business changes
    * latency
      * Read should be fast, within one seconds
* functional and non-functional features
  * Functional
    * GetBusiness(longitude, latitude, radius) : {pagination, count: {}, data: [{}{}...]}
      * `POST v1/business/search`
    * IndexBusiness(business)
      * `POST/GET/PUT/DELETE v1/business/{:id}`
    * Non-functional
      * Availability, low-consistency
      * Scalability 
      * Fault-tolerant
* Domain and data model
  * what data we have
    * Business metadata
      * Business name/id, location (lat,long), country, state, province
    * We need to aggregate data and index them for fast search queries
  * What database to use
    * Business table for business CRUD and to render the result page (it can be a NoSql db)
      * 50 million business * (1kb/business) = 50 GB
      * 100 million DAU * 5 queries/user / 100,000 sec/day = 5000 QPS 
    * Search table to search queries (geospatial indexed table)
      * `Location, businessId, lat, long, country, state` (2 + 8 + 8 + 8 + 8 + 8) = 40 bytes
      * 40 bytes * 50 MM business = 40 * 50 * 10^6 = 2000 * 10^6 = 2 * 19^9 = 2 GB (tiny, fly weight)
      * we could use in-memory, disk
  * How to process data
    * Business owner index data into table, this can be done in async (pull model) (data ingestion)
    * Because our service is read heavy, we need design to scale up for read
      * Shard/replica/cache, which one to use depends on the size of our data storage
* Geospatial Index algorithm
  * Common pain point
    * lat and long is two dimensions, common query language is slow when doing query on 2 dimensions, cuz it results in a table scan
      * Use long + lat as compound key, is not really possible as it's an intersection point, not a zone,
    * solution is to map 2-dimensional data into 1 dimensional, and therefore improve the query efficiency (logarithm)
  * Hash based
    * Even-grid: divided 2-dimensional map into evenly distributed grids. Problem is business is not evenly distributed in each grid
    * Geohash
      * Use 00, 01, 10, 11 to divide map into 4 grids
      * then for each grid, do further division for 00 -> {00 00, 00 01, 00 10, 00 11} 4 bits
      * encode bits into base32 string for e.g. `1001 10110 01001 10000 11011 11010` -> `9q9hvu`
      * since geohash is string, and one property of string is query can be done in any database, we don't need a geospatial db
      * Use precision to define the grid size in real world (km * km)
      * there is table explaining the relation between geohash length (in encoded string) and the grid size
        * ![](https://github.com/Danny7226/LearningSummary/blob/main/systemdesign/assets/proximity/table.png)
        * Usually we store the level 6 base32 encoded string as Geohash key, and pay attention to level 4-6 (20km - 600m in radius) when doing the query,
        * As prefix characters defines the large area whereas the following character specifies which sub-area it is in the large scope
          * `9q9hv*` defines the (4.9km * 4.9km square)
          * Similar prefix means location is close, but reverse is not true (location is close, prefix is similar)
        * when doing the query, choose the right level (choose grid level whose granularity covers the whole circle)
          * 500m radius, do level 6 search, query all 8 neighbours of `9q9hvu`
          * 4km radius, do level 5 search, query all 8 neighbours of `9q9hv`
          * Calculating neighbours in geohash can be provided by lib in constant time
          * `SELECT * from input_data where state = 'WA_US' and substr(geohash,1,6) IN ('w21zd2', 'w21z6r', 'w21z6q', 'w21z6m')`
  * Tree based (in memory storage, different set of deployment strategy and operational requirements)
    * Guadtree
    * Google s2
    * Rtree
    * A trie might be good for search if it's fly weight (can be loaded into RAM), TODO give it a further thought
      * [ref](https://blog.mapbox.com/a-dive-into-spatial-search-algorithms-ebd0c5e39d2a)
      * Tree depth is `ceil(log(numberOfGrids) / log(divisionOfLevel))`. For e.g. 1MM grids with 9 divisions per level, depth is 7
      * Range search takes `depth * divisionOfLevel` comparisons. 7 * 9 = 63 for 1MM grids with 9 divisions per level
      * K-th closest neighbours search requires a data structure priority queue, based on the rank of the distance of targeting grid (segment-point and segment-box distance)
        * Putting tree nodes from the highest level, and unbox and put in children of the closest one in the PQ
        * Keep doing and add nodes to the return list if it is of the targeting level
        * Complexity is O(k * depth * divisionOfLevel * PQ_Insertion)
* System diagram
  ![](https://github.com/Danny7226/LearningSummary/blob/main/systemdesign/assets/proximity/system.png)

### System design case 3: Amazon Phone Tool service
be able to show upper-chain managers and employee's direct reports
be able to display basic employee information (tenure, name, department, role, position level, etc)
1 days delay for write
tens of milliseconds latency for read

### System design case 4: review with pagination
* Clarification for ambiguity
  * Customer
    * Shoppers who want to leave a review for a product
    * Shoppers who want to see reviews of a product
    * Business owners who want to manage products of their business store
  * Scale
    * 1MM business owners
    * 100 products per business
    * 1000 MM daily active shoppers
    * 1% of DAU leaves a review
  * Performance
    * relatively high acceptance criteria for write-to-read delay
    * reviews read should be fast, within order of magnitude of milliseconds
    * highly available, fault-tolerant, but doesn't have to have strong consistency
* Functional and non-function requirements
  * be able to write reviews for a product
    * `v1/reviews` POST
  * be able to retrieve reviews for a product with pagination based on different rank (time, star, time and star)
    * BATCH GET `v1/reviews?page=;sort=;` with URL param such as pagination offset/token, page size, sort filter
  * be able to keyword search review
    * BATCH GET `v1/reviews/search?product=;keyword=`
  * be able to view products or a business
    * `v1/business/{:id}`
    * `v1/products/{:id}?business=id;`
  * Non-functional as in ambiguity performance section above
* Data modeling
  * Entity, ER, 
    * Reviews
      * reviewId, productId, reviewTitle, review content, review BLOBid (1KB)
    * Business
      * business id, ownerId , business description (100B)
    * Product
      * productId, businessId, productName, product description, product price (100B)
  * size of storage
    * Product table: 1MM business owner * 100 products per business * 100B per product = 1 * 10^10B = 10GB small size SQL
    * Business table: 1MM business * 100B = 100 MB fly weight
    * Reviews table: 1% * 1000MM DAU * 1KB/review = 10GB / day, 4 TB / year
  * TPS: 
    * write reviews 10MM / 100,000 sec/day = 100 TPS
    * read reviews 1000MM / 100,000 sec/day = 10k TPS
  * Database schema
    * Since there is strong relationship between business and products, we will use SQL for business and product table
    * Since data volume of reviews is huge and access pattern is straightforward and ready heavy and latency needs to be low, we will use NoSQL
    * Business table
      * BusinessId (pk), ownerId(fk), business info columns...
    * Product table
      * ProductId (pk), businessId (fk), product info columns...
    * Reviews table (support sorting, and filtering by star feature)
      * Partition key: productId `uniqueProductId`
      * Sort key: timestamp_reviewId `1700592636_uniqueReviewId`
      * GSI: star_productId `3star_uniqueProductId`
      * Attributes (key-value pair): reviewTitle, reviewContent, reviewBlobs
    * Reviews search table (inverted index table)
      * keyword: keyword_string
      * reviewIdentifier: productId_timestamp_reviewId
* System diagram
  * ![](https://github.com/Danny7226/LearningSummary/blob/main/systemdesign/assets/review/system.png)
  * CRUD of business and product will be through biz & product proxy and interact with SQL DB directly
  * Write of reviews will go through data ingestion pipeline first 
    * It will smooth the traffic, provide failure recovery and process at its own space
    * ingestion queue can be partitioned and replicated in different data center to improve processing speed as well as fault-tolerant
    * reviews data will be stored in Reviews NoSQL db as well as inverted-index db for keyword search
  * Read of reviews will interact with ReviewsDB
    * reviews data are sorted based on timestamp
    * normal pagination will leverage Sort key: timestamp_reviewId
    * star filter will look up `star_productId` partition nodes along with sort key: timestamp_reviewId

### System design case 5: Monitoring JVM metrics 
Develop a system to collect JVM metrics data within Docker centers. Display this data on a user interface every 12 hours to monitor the server's remaining lifespan. 
The JVM instances in the Docker center should be able to communicate their health status, wherein health is perceived as a value diminishing with wear and tear. 
Implement immediate notifications for both JVM failures and instances where health data falls below a predefined threshold.
* every 12 hours, means our data storage doesn't need to store real time data
* notification immediately means it needs to be real time

* Ambiguity
  * Who is our customer
    * person that subscribe to data server notifications
    * person that monitor data server remaining life
  * Func
    * getDataCenterDetails
    * subscribe(dataCenter)
  * Scale
    * 100MM servers reside in 1MM data centers
    * 1 million subscribers
    * low read TPS, 10 - 100 TPS read with cache
  * Non functional requirements
    * Highly available
    * Scale-able
* data model
  * raw data we have
    * `dataCenterId` `dataServerId` `remainingLife`
    * `dataCenterId` `subscriberEndpoint`
  * No need for strong consistent ACID transactions so NoSql database
* System diagram
  * How do we get the data
    * Since don't need to present data in real time, emit health data every minute
    * Need to have a mechanism to check heartbeat of data server (either gossip or a master node)
  * How to process data
    * Since monitoring data doesn't need to be real time, we put raw data in a data lake, which is append-on log files
    * A ingestion pipeline with a partitioner is needed in order to partition raw data into smaller portion 
    * Works can process data lake data and aggregate them for a certain period of time (store in memory first), and then write the health DB
    * If health below threshold, notify 
    * After data aggregation, safely delete data in data lake
    * Gossip or master node is used to check if a host is dead, if dead, send notifications immediately
  * ![](https://github.com/Danny7226/LearningSummary/blob/main/systemdesign/assets/datacenterhealth/system.png)

* Round2 recap
  * Since we don't need to present result in real time, only every 8 hours, we could 
    * have let the data servers send healthy info to a stream with partitioner, 
    * an in-memory compute engine reads each partition data and update the in-memory table (k-v map)
    * if in-memory compute engine dies, simply replace with another one since we don't need to memorize the previous state, no recovery cost
      * if need to memorize, have SSTable and commit log in disk to recover
    * Every couple of hours, update the main table for monitor reading purpose (or simply read data directly from compute engines)
    * ![](https://github.com/Danny7226/LearningSummary/blob/main/systemdesign/assets/datacenterhealth/system2.jpg)
  
  
  