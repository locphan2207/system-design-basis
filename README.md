# System Design Basis
### CAP theorem:
- _Consistency_: All nodes must see the same exact data
- _Availability_: The system must be available. Requests must receive response (success/failure).
- _Partition Tolerance_: The system must continue to work despite wrong partition.
A system can only guarantee either consistency or availability. 

#### Consistency:
- _Weak consistency_: It's allowed to lose a part of data, but when it's back, it resumes with the most recent write. (Ex: VoiD)
- _Eventual consistency_: Data will eventually be sent. Good for high availability system. (Ex: email)
- _Strong consistency_: Data must be seen right after write. (Ex: RDBMS)

### Redundancy and replication:
- _Active-passive_: Replicate. In case of failure, do a failover to the passive one. The passive one listens to the heartbeat sent from the main in order to know when it needs to replace. Downtime is the time when the passive ones boots up.
- _Active-active_: Both manages traffic concurrently by spreading it.
- _Master-slave_: For DBMS, slave copies data from master and usually will serve reads. Master will do writes. If master dies, system can be read-only
- _Master-master_: For DBMS, both masters serve reads and writes. They have to sync each other to be consistent. If one dies, the other can take over.

### DNS (Domain Name System):


### Load Balancer:
- Help distribute incoming traffic.
- Avoid unhealthy servers
- Prevent overloading servers
- Important for horizontal scaling, avoid single point of failure.
- Can also decrypt resquests and encrypt responses.
- Can store session cookies
- Methods: random, weighted/non-weighted round robin, layer 4, layer 7, IP hash, least bandwidth, least response time, least connection.

### Cache:
- A short-term memory
- Help store a response/data in a period of time with the believe that recent requests can be asked again
- Could exist in different places in a system 
- Should invalidate/update cache if there is a corresponding change in db:
  - Write-through: From the request that changes the data, update it in the cache first, then update to db, then return response to client. High latency, but data is consistent and fresh.
  - Write-back: Update the the cache only. Only update the db after a time interval. Low latency, but can result in data loss if cache dies before storing to db
  - Write-around: Update the db only. Can create a "cache miss".
1. CDN (Content Delivery Network):
- A type of cache.
- Help serve content by choosing the closest server of the client location.
- If a system has CDN, the application servers don't have to send things that are already in CDN.
- If don't have money to make stand-alone CDN server, use Nginx.
2. Application server cache:
- Store response of every recent requests.
- Use global cache or distributed cache if there are many servers (horizontal scale)

### Partitioning:
1. Methods:
- Horizontal partitioning (sharding): Put different rows in different tables such as range based partitioning (ex: split name into range of alphabet a-d, e-f, etc). Should carefully chose how to split the data to have balanced tables.
- Vertical partitioning: Put different tables but related to a server together. (ex: tables related to user to a server)
- Directory based partitioning: Make a lookup service that holds the mapping of the partitions. Can query to this service to find out where/which partition to go to.
2. Criteria:
- Key of hash based partitioning: Apply hash function to get the id of the partition. Could use modulus on the number of the partitions. This could results in downtime when adding new parition. Consistent hashing could be a solution in that case.
- List partitioning: Each partition contains a list of keys which will be used to determine where data will be store. Ex: partition 1 stores houses in California, Utah.
- Round-robin partitioning: A simple strategy to make uniform partitions.
- Composite partitioning
3. Common problems:
- Joins and denormalization: Joins will not be fast since it has to collect data from partitions from different machines. Denormalization can be a quick solution for this, so it doesn't have to do many joins. However, this can create data inconsistency.
- Referential integrity
- Rebalancing: Have to rebalance if a partition experiences overloading issue or they are not uniform. Rebalance is usually resulted in downtime as it has to move data from partitions to different servers. If a system uses directory based partition, it might be possible to do without downtime, but it can create big complexity and also the directory server could be a single point of failure.

### Indexes:
- Useful to increase read performance, but reduces write performance because it has to update indexes 
- Usually means keep data sorted with keys and store the pointers to the beginning section of each key for fast access.

### SQL vs NoSQL:
1. SQL (Relational db):
- Structured
- Data is stored in table. Rows are data set. Columns are data points.
- Good for vertical scaling, challenging to do horizontal scale.
- Good for ACID (Atomicity, Consistency, Isolation, Durability)
- Could be expensive in scaling.
2. NoSQL (Non-relational db):
- Unstructured, dynamic.
- Could be key-value (DynamoDB), document (MongoDB), wide-column (Cassandra) or graph.
- Good for horizontal scaling, 
- Violate ACID.
- Affordable to scale, can store PB of data with low cost.
3. Reasons for SQL:
- ACID compliance. Strict schema
- Data structure doesn't change too much
- Have complex joins, have a lot of relationships.
- Look up by indexes is fast
- Well established
4. Reasons for NoSQL:
- Data doesn't have many relationship.
- Flexible schema
- No complex joins, not many relationships.
- Rapid development since it is easy to scale horizontally
