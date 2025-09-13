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
Automation is possible and increasingly common, but manual sharding is still used for custom needs.
