### System Design
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