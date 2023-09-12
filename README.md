# LearningSummary

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
