# System Design Fundamental Concepts

## 1. Distributed system

Network of computers working as one coherent system

### Characteristics

- **Scalability**: the system’s ability to handle growing demands. Two types of scaling. Horizontal scaling is by adding more servers. Vertical scaling by upgrading existing hardwares.
- **Reliability**: the system’s ability to function correctly even when components fail.
- **Availability**: the percentage of time the system remains operational.
- **Efficiency**: measured by two main factors. **Latency**; delay in getting the first response. **Throughput** which is the number of operations handled in a given time.

These characteristics often involve tradeoffs. The goal is to balance these factors based on the given requirements.

### CAP theorem

A distributed system can only guarantee two out of the three properties.

- **Consistency**: all nodes display identical data, guaranteeing that reads reflect the most recent writes
- **Availability**: every request always receives a response without guaranteeing that it reflects the most recent write
- **Partition tolerance**: the system continues to function despite network failures between nodes

## 2. API Gateway

### Main three tasks:

- Route requests
- Authentication
- Rate limiting

## 3. Load balancer

Distributes incoming requests evenly across servers to ensure that no single server becomes overwhelmed. If one server goes down, the load balancer will only direct traffic to healthy servers.

## Some algorithms for load balancer to distribute traffic

- **Least connection method**: send the connection to the server with the fewest active connection
- **Round robin**: cycle through a list of servers sequentially
- **IP hash**: use he client’s IP address to determine which server to send to.

Note that LB itself could be a single point of failure. To prevent this, we add another LB for a standby. So that if the primary one fails the second one takes over immediately.

## 4. Queue

Queue decouples requests from potentially slow processing work. Benefits include:

- API responds quickly
- Workers can process jobs asynchronously
- Can scale workers independently of the API
- Temporary spikes are absorbed by the queue
- Failed jobs don’t necessarily mean that the original request/upload failed

The worker only acknowledges the job and removes from the queue only after successful processing:

1. Worker receives job.
2. Worker starts processing.
3. Worker crashes or times out
4. Queue notices the job wasn't acknowledged
5. Job becomes available again.
6. Another worker retries it.

We retry jobs automatically with exponential backoff, for example:
Attempt 1 -> fail -> wait 1 minute
Attempt 2 -> fail -> wait 2 minute
Attempt 3 -> fail -> wait 8 minute
Attempt 4 -> fail -> wait 16 minute

There are two types of failures we should distinguish:

- **Transient failures**: network errors, temporary service outage -> retry
- **Permanent failures**: corrupt files, unsupported format -> don’t retry indefinitely

After a maximum number of attempts, move the job to a dead-letter queue so it can be inspected.

How do we avoid processing the job twice?
Design the processing operation to be idempotent. Give every upload a unique ID and keep a database record for the status. Before doing the work, the worker checks if the upload has already been successfully processed. The important idea is that the queue provides at-least-once delivery so the job handler must tolerate duplicate delivery.

## 5. Caching

If we start noticing that the servers request the same data, that's where caching comes into play. It takes advantage of the fact that the recently requested date is likely to be requested again. Retrieving data from cache is much faster than retrieving from the original db.

### CDN

Aside from application cache, there is also Content Delivery Network which is ideal for serving static media. CDN cache content closer to the user to reduce latency.

### Caching challenges

Maintaining data consistency, making sure that data is in sync with the source of truth. We don't want to serve the data from cache if it's not up to date.

### Some invalidation strategies:

- **Write through**: data is written to cache and storage at the same time. Increase consistency, but increase write latency.
- **Write around**: data bypasses cache and goes directly to the storage. Preventing cache flooding, but increase read latency
- **Write back**: data is written to cache first and later to storage. Low latency, but risk of data loss in case of system failure

### Eviction policy:

When cache reaches its capacity, we need an eviction policy:

- **Least recently used** (LRU): Removes the least recently accessed item.
- **First in first out** (FIFO): Removes the oldest item.
- **Least frequently used** (LFU): Removes the least frequently used item.

## 6. Storage strategy

TODO: add something about ObjectStorage

As our platform grows, we need to think about storage strategy.

SQL vs NoSQL?

SQL db:
Rational db
stores data with predefined schemas.
Each row contains information about a piece of record.
If you want to add a new column, the change would have to be applied to all the records in the table.
(e.g., oracle, mysql, postgresql)

NoSQL db:
non-relational db.
More flexible data structure.
Four main types. Key value stores (redis), document db (mongo db), wide column stores (cassandra), graph db (neo4j).

SQL
NoSQL
Rigid schema
Flexible schema
SQL language
Collection of documents
Typically scales vertically, but can scale horizontally through sharding
Horizontal scaling
ACID compliant
Not ACID compliant, but performance and scalability

### ACID

- **Atomicity**: the transaction is fully completed or not at all
- **Consistency**: a database transaction guarantees that the database is taken from one valid state to another enforcing all the predefined rules
- **Isolation**: keeps transactions separate so their operations do not interfere with each other
- **Durability**: once the transaction is completed, it remains permanent even in case of failure.

Use SQL if we need ACID compliance (e.g., financial applications) and our data structure does not change often, structured, and consistent.

Use NoSQL if we are dealing with large volumes of unstructured data, need flexibility, and rapid development.

## 7. Data query performance

### Indexing

No indexing means that we have to search through the entire user's table every single time. Indexing works by creating a separate data structure that points to the actual data to speed up the search operations. Mainly speeds up reads.

#### Common indexing strategies:

The most common indexing targets are primary indexes,
Secondary index non primary key columns like firstname.
Composite indexing is created on multiple columns useful for queries those columns involved together like firstname, lastname, and email.

It depends on the query pattern.

Foreign keys ensure relationships between columns and different tables.

Tradeoff:
It can improve read performance, it can slow down write performance. This is because every time you insert, update, or delete data the index must also be updated.

### Partitioning

When our database can no longer scale vertically, we can look into data partitioning. It's a technique to break large databases into smaller, more manageable parts. This improves performance, availability, and load balancing. Mainly speeds up reads and writes, and helps scalability.

### Partitioning methods:

- **Horizontal partitioning** (sharding): divides rows of table across multiple databases
- **Vertical partitioning**: separates entire columns or features across multiple databases
- **Directory-based partitioning**: uses lookup service to abstract a partitioning scheme

Partitioning technique can be done key or hash based, determining which database to store based on the hashing of the data.

Techniques:
Consistent hashing when scaling the number of servers. Only a small portion of data needs to be remapped so make it easier to scale.

List partitioning: assign each partition a list of values, storing each data which the list its key belongs to.

Round robin: distribute the data evenly across partitions in a circular order
Composite partitioning combines two or more partitioning methods.

Partitioning challenges:
Difficulty joining across multiple partitions. Leading to potentially tricky data reconciliation.
