## Index
[Area of focus]()

[Reversed index](https://github.com/Danny7226/LearningSummary#reversed-index)

[Search service](https://github.com/Danny7226/LearningSummary#search-service)

[Data processing pipeline](https://github.com/Danny7226/LearningSummary#data-processing-pipeline)

## System Design
### Area of focus
* Ask for clarifications
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
  * non-functional requirements are ones like fast, fault tolerant, SLA, secure

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
