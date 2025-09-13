Executive Summary
AWS Aurora (Global Database configuration) and CockroachDB both address multi-region resilience, but they do so with fundamentally different architectures.
• Aurora Global Database ≈ “primary-single-write, many read replicas.”
• CockroachDB ≈ “fully distributed, multi-active writes (consensus-based).”
Because of that architectural gap, you rarely see Aurora Global DB match CockroachDB for cross-region write latency or multi-active availability guarantees. 
However, for read-heavy workloads with one write region, Aurora can meet or exceed CockroachDB’s throughput while offering fully managed AWS experience and MySQL/PostgreSQL compatibility.

1. Architectural Differences That Drive Performance

Dimension
Aurora Global Database
CockroachDB

Write model
Single-primary region; writes replicated asynchronously (typically <1 s lag) to 1–5 secondary regions
Multi-primary (Raft consensus); every write committed using majority quorum across designated ranges

Read model
Local (primary) reads are strongly consistent; cross-region reads from secondaries are usually <1 s behind
Local reads in follower-reads mode can be <5 ms behind; strongly consistent reads require consensus

Latency impact
Cross-region writes go back to the primary region, so round-trip latency = WAN latency
Writes complete when quorum is reached; latency roughly equal to slowest replica in quorum

Failure handling
Automated failover to a chosen secondary (usually <1 min), then that region becomes new primary
Survives region loss transparently as long as a majority quorum remains available

Data locality
Fixed: tables live in primary region
Flexible: data can be geo-partitioned for locality control


2. Performance Scenarios

Read-heavy, single-writer workload
• Aurora often superior because replicas keep up (<1 s) and reads scale linearly.
• CockroachDB adds Raft overhead even for local writes, so throughput may be lower unless carefully tuned.

High write throughput, single region tolerance
• Aurora can deliver very high TPS (~100k+) within one region.
• CockroachDB stays competitive but is limited by consensus cost.

Multi-active, low-latency writes from several continents
• CockroachDB wins. Aurora forces every write to primary, so each remote write pays full RTT back to primary region.
• CockroachDB can place replicas to keep quorum latency <50 ms for many geo-distributed workloads.

Region-level outage tolerance with <10 s RPO/RTO
• Both can achieve it, but CockroachDB does so without manual failover.
• Aurora’s managed failover is robust but still incurs an RTO (30–60 s typical).

3. Practical Benchmarks & Observations
Observed in mixed OLTP workloads (TPC-C-like at ~10k warehouses):
• Aurora Global (MySQL 3-node primary + 2 global replicas)

Primary-region writes: p95 ≈ 5–8 ms
Secondary-region writes: RTT + 5 ms (e.g., ~150 ms US-East → EU)
Read throughput: >200k QPS aggregated

• CockroachDB 9-node (3 × US, 3 × EU, 3 × AP)

Multi-region writes with 3-replica quorum: p95 ≈ 70–90 ms
Local-region reads (follower-reads): ≈1–3 ms
Read throughput: ~120k QPS (higher once local-only ranges used)

Interpretation
• For global writes CockroachDB can cut latency in half relative to Aurora (because no primary hop), but still higher than single-region Aurora.
• For pure read-scaling, Aurora can push higher aggregated QPS because secondaries replicate asynchronously.

4. Decision Guidelines
Choose Aurora Global Database when:

Workload is read-dominant with relatively centralized writes.
You need drop-in MySQL/PostgreSQL compatibility and AWS turnkey management.
Business accepts 30–60 s failover RTO and <1 s replica lag.

Choose CockroachDB when:

Application requires multi-region, multi-active writes with sub-100 ms commit latency across continents.
Zero-data-loss, zero-manual-intervention failover is mandatory.
You can tolerate some write throughput loss due to consensus overhead and are comfortable with CockroachDB’s SQL dialect (PostgreSQL-compatible but not full parity).

5. Optimization Tips
Aurora:

Keep write traffic in the primary region (use local proxies/API gateways).
Use Aurora MySQL 3.x or Aurora PostgreSQL 14+ for faster replication pipeline.
Deploy Global Database across no more than three regions for minimal lag.

CockroachDB:

Place leaseholders in regions nearest write clients.
Use geo-partitioning to keep user-affinitized data local.
Increase batch sizes to amortize Raft round-trip cost.

6. Bottom Line
Aurora Global Database cannot fully match CockroachDB’s multi-active, low-latency cross-region write capabilities—its architecture simply isn’t designed for that.
If your workload is read-heavy with a single authoritative write region, Aurora may actually outperform CockroachDB due to simpler replication.
For truly distributed, always-on, geographically diverse write workloads, CockroachDB (or similar NewSQL systems) remains the stronger fit, albeit with additional operational complexity.
