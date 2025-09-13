Manual Sharding vs Automatic Sharding

CockroachDB, DynamoDB, Azure CosmosDB, Google Cloud Spanner all support automated sharding.

What's manual sharding of databases, why organizations might do it, and whether automation is possible or advisable.

What Is Manual Sharding?
Sharding is a technique used to split a large database into smaller, more manageable pieces called shards. Each shard holds a subset of the data, often based on a key like user ID or geographic region. This helps with scalability, performance, and sometimes fault tolerance.

Manual sharding means that the database administrators (DBAs) or developers themselves design, implement, and manage the logic for splitting data across multiple databases or servers. 
This includes:

Deciding how to partition the data (e.g., by customer ID ranges, regions, etc.)
Writing code to route queries to the correct shard
Handling cross-shard queries and transactions
Managing schema changes across shards

Example Scenario
Suppose you have a user database with millions of records. You might manually shard by splitting users into separate databases based on their country:

US users in one database
EU users in another
Asia-Pacific users in a third

Your application code must then know which database to query based on the user's location.

Can Sharding Be Automated?
Yes, sharding can be automated—but with caveats.
Automated Sharding
Many modern databases and cloud services offer automatic sharding (sometimes called "auto-sharding" or "horizontal partitioning"). Examples include:

MongoDB: Built-in sharding capabilities
Google Cloud Spanner, Amazon DynamoDB, CockroachDB: Automatically distribute data across nodes

With automated sharding, the database system itself handles:

Data distribution and balancing
Routing queries to the correct shard
Rebalancing shards as data grows or shrinks

Why Manual Sharding Still Exists
Despite automation, manual sharding is sometimes preferred when:

The database technology in use does not support automatic sharding
The organization needs custom logic for partitioning (e.g., business-specific rules)
There are performance or compliance requirements that require precise control

Manual sharding can offer flexibility, but it comes with complexity and operational overhead.

Practical Implications and Considerations

Manual sharding increases development and maintenance effort. You must handle edge cases, migrations, and failures yourself.
Automated sharding reduces operational burden but may have limitations (e.g., less control over data placement, potential for "hot spots").
Hybrid approaches exist, where initial sharding is manual, but rebalancing or scaling is automated.

Risks and Limitations

Manual sharding can lead to uneven data distribution ("hot shards").
Schema changes and cross-shard queries are harder to manage manually.
Automated sharding may not fit every use case, especially with legacy systems.

Summary

Manual sharding: You design and manage the partitioning logic yourself.
Automated sharding: The database system handles partitioning and routing.

--------------
Traditional sharding: Most databases (like MySQL, MongoDB, etc.) use sharding by splitting data into large, manually managed chunks (shards), each assigned to a specific server. 
Application logic or middleware often decides which shard to query.
CockroachDB’s approach: CockroachDB automatically divides tables into small, dynamic ranges (typically ~64MB each). 
These ranges are distributed across the cluster and moved as needed for load balancing, fault tolerance, and scaling.

How CockroachDB Achieves Horizontal Scaling

Range-based partitioning:
Each table is split into many ranges, which are the unit of distribution and replication.
Ranges are automatically split, merged, and rebalanced as data grows or access patterns change.


Replication & consensus:
Each range is replicated (usually three copies) across different nodes using the Raft consensus protocol.
This ensures high availability and consistency.

Automatic rebalancing:
When you add more nodes, CockroachDB automatically redistributes ranges to balance storage and query load.
No manual sharding or rebalancing required.

Horizontal Scaling
Add nodes: You can add more nodes to the cluster, and CockroachDB will automatically spread data and workload across them.
Remove nodes: You can also remove nodes, and CockroachDB will rebalance data to maintain redundancy and performance.

Vertical Scaling
Upgrade hardware: You can increase CPU, memory, or disk on existing nodes for better performance.
Limitations: While vertical scaling helps, CockroachDB’s real power is in horizontal scaling—adding more nodes for more capacity and resilience.

Elastic Scaling is the ability to scale resources up or down dynamically as demand changes.
In CockroachDB:

Primarily refers to horizontal scaling—nodes can be added or removed with minimal manual intervention.
Vertical scaling is supported but less impactful compared to horizontal scaling for distributed workloads.

Why This Matters

No manual sharding: Simplifies operations, reduces risk of “hot spots,” and makes scaling seamless.
True elasticity: CockroachDB can handle traffic spikes or growth by simply adding nodes—no downtime or complex migrations.
Resilience: Automatic rebalancing and replication mean the system can survive node failures without data loss or service interruption.

--------------
Single Shard distribution:

A single, independent database shard cannot span across different data centers and regions in the cloud. This is because sharding is a horizontal partitioning strategy that distributes distinct data subsets across separate physical database servers or nodes. These nodes must be logically separated to achieve the benefits of sharding, such as increased scalability, improved performance, and enhanced fault isolation. 
Attempting to stretch a single shard across multiple data centers or regions would negate these benefits and introduce significant technical and operational problems.

Key reasons a single shard must be confined:

Performance and latency: A key benefit of sharding is placing data physically closer to users through geo-sharding to reduce latency. Placing parts of a single shard in geographically distant locations would introduce high network latency for any operation involving more than one part of that shard. This would drastically degrade performance and defeat the purpose of geographic distribution.

Logical vs. physical isolation: Sharding is designed around a "shared-nothing" architecture, where each shard operates independently. A single shard represents a logical subset of the data, and its physical manifestation (the server or node) must be in a single location to function as an autonomous unit.

Data consistency and transaction complexity: Ensuring data consistency and integrity (ACID properties) for a single transaction across different physical locations is extremely challenging and adds significant overhead. A cross-region transaction would introduce immense latency, require complex coordination, and make the database vulnerable to network failures.

Availability and fault tolerance: Sharding improves fault tolerance by isolating failures to individual shards. If one shard fails, others continue to operate. Forcing a single shard to span regions would make it a single point of failure. A regional network outage could corrupt or disrupt the entire shard instead of just affecting the local data center. 

How multi-region sharding is actually implemented
To achieve global scale, databases use replication in combination with sharding, not by stretching single shards. For example, a geo-sharding strategy would work as follows: 

Logical partitioning: The overall database is logically partitioned by region.

Shard per region: All data for the Europe region is stored in a shard located within a European data center. Similarly, data for North America is stored in a separate shard within a North American data center.

Cross-regional replication (for disaster recovery): Each regional shard can have its own replicas in other regions to serve as a backup for disaster recovery. The database system handles the synchronization of data between these regional replicas.

Automatic routing: A routing layer within the application directs user requests to the correct regional shard to minimize latency. 

Cloud providers like Oracle, AWS, and Azure offer distributed database solutions (such as Google Cloud Spanner, Azure Cosmos DB, and Oracle Sharding) that automate and manage this complex process, allowing for horizontal scaling and global data distribution across multiple regions while maintaining data consistency. 
Automation is possible and increasingly common, but manual sharding is still used for custom needs.
