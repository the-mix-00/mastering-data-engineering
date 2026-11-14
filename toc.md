# Complete Study Guide: Junior → Senior Data Infrastructure Engineer

> **How to use each subsection:** Paste the subsection number/title into a fresh AI chat using the Master Prompt.
> * **Modules 1, 4–12:** Use the 5-step prompt as-is.
> * **Modules 2–3:** Ask for a *15-line "slow/broken vs. fast/safe" Java or Python snippet*.
> * **Modules 13–14:** Swap memory/disk layout for *a realistic flame graph, benchmark output, or cluster config artifact*.
> * **Module 15:** Run as *interactive paper walkthroughs (15.1–15.4)* and *live Staff system-design mock interviews (15.5–15.6)*.

---

## Phase 1: Systems & Language Foundations *(Junior → Strong Mid)*

### 1. Hardware, OS & Memory Mechanics
* **1.1 CPU Caches & Memory Locality:** How L1/L2/L3 cache lines, prefetching, and branch prediction work—and why sequential array scans beat pointer-chasing by 10–100x.
* **1.2 Virtual Memory, Page Cache & Zero-Copy I/O:** Page tables, OS page cache, `mmap`, buffered vs. direct I/O, and zero-copy paths (`sendfile`, `io_uring`) that avoid redundant CPU copies.
* **1.3 SIMD & Vectorized CPU Execution:** How CPUs process 4–16 values in one instruction (AVX2/AVX-512/NEON) and how data alignment unlocks hardware-speed filtering and math.

### 2. Systems-Grade Java & High-Performance Python *(Include 15-line slow vs. fast snippets)*
* **2.1 JVM Internals, Object Overhead & GC Tuning:** Heap layout, object header bloat, allocation rate bottlenecks, and tuning G1GC/ZGC to eliminate multi-second pauses.
* **2.2 Off-Heap Memory & Java Native/Vector APIs:** Managing raw contiguous buffers outside the GC (`ByteBuffer`, Project Panama Foreign Memory API, and Java Vector API).
* **2.3 High-Performance Python & Native Bridges:** Why pure Python loops choke at scale, GIL mechanics, zero-copy NumPy/PyArrow buffers, and exposing C++/Java engines cleanly to Python users.

### 3. Concurrency & Network Programming *(Include 15-line slow vs. fast snippets)*
* **3.1 Shared-Memory Concurrency, Locks & Contention:** Thread pools, mutex/RW-lock contention, false sharing across CPU cache lines, and work-stealing schedulers.
* **3.2 Lock-Free Patterns, Atomics & Memory Models:** Compare-And-Swap (CAS), single-producer/consumer ring buffers (LMAX Disruptor), and memory visibility barriers (`volatile`/happens-before).
* **3.3 Async Networking, Backpressure & Serialization Cost:** Non-blocking I/O (`Netty`), TCP flow control/backpressure, gRPC vs. Apache Arrow Flight, and why JSON/Protobuf serde dominates CPU budgets.

---

## Phase 2: Storage Engines & Columnar Formats *(Mid-Level Core)*

### 4. Database Storage Engine Internals
* **4.1 Row-Oriented Engines: B-Trees vs. LSM-Trees:** How OLTP engines handle point lookups vs. high-velocity writes, Write-Ahead Logs (WAL), write/read amplification, and LSM compaction.
* **4.2 Buffer Pools & MVCC Snapshot Isolation:** Managing pinned memory pages, dirty-page flushing, and Multi-Version Concurrency Control so analytical readers never block writers.

### 5. Column-Oriented Storage & In-Memory Formats
* **5.1 Row vs. Columnar Layouts & Access Patterns:** Why reading 3 columns across 10 billion rows flies in columnar stores—and why single-row updates/deletes are painful.
* **5.2 Columnar Compression & Encodings:** Dictionary encoding, Run-Length Encoding (RLE), bit-packing, delta/frame-of-reference encoding for timestamps/prices, and executing filters *directly on encoded data*.
* **5.3 Apache Parquet File Internals:** Row groups, column chunks, pages, footer metadata, min/max zone maps, and Bloom filters for skipping 99% of disk reads.
* **5.4 Apache Arrow In-Memory Format & Zero-Copy IPC:** Standardized columnar memory buffers, validity bitmaps, offset arrays, and passing data between Java, Python, and C++ with zero serialization.

### 6. Cloud Object Storage & Lakehouse Table Formats
* **6.1 Object Store Physics (S3/GCS):** High throughput paired with high first-byte latency (~50–100ms), range-read coalescing, prefix limits, and local NVMe caching tiers.
* **6.2 Open Table Formats (Apache Iceberg / Delta Lake):** Bringing ACID snapshots over immutable Parquet files via manifest trees, metadata pruning, schema evolution, and fixing the "small files problem."

---

## Phase 3: Query Engine & Execution Internals *(Mid → Senior Bridge)*

### 7. Anatomy of an Analytical Query Engine
* **7.1 Parsing, Catalog Binding & Query Plans:** Turning SQL into ASTs, resolving schemas/catalogs, building logical relational algebra trees, and lowering them to physical operator plans.
* **7.2 Execution Models (Volcano vs. Vectorized vs. JIT):** Row-at-a-time pull iterators vs. batch vectorized execution (DuckDB/ClickHouse/Velox) vs. runtime LLVM/JVM codegen (Spark/Hyper).

### 8. Core Operators & External Algorithms
* **8.1 Filtering, SIMD Bitmasks & Late Materialization:** Evaluating predicates into selection bitmasks and deferring reads of unused columns until surviving row IDs are known.
* **8.2 Partitioned Hash Joins & Broadcast Joins:** Build vs. probe sides, cache-conscious hash tables, Bloom-filter pushdowns across joins, and deciding when to broadcast small dimension tables.
* **8.3 Sort-Merge Joins & Financial ASOF Joins:** Streaming sorted joins for massive tables, and **ASOF joins** for aligning asynchronous time-series events (e.g., matching trades to the latest prior quote).
* **8.4 Aggregations, External Merge Sort & Disk Spilling:** Hash vs. sort `GROUP BY`, approximate sketches (HyperLogLog/t-digest), external multi-way merge sort, and spilling gracefully to NVMe before hitting OOM.

### 9. Query Optimization
* **9.1 Rule-Based Optimization (RBO):** Constant folding, predicate pushdown into Parquet/Iceberg stats, projection pruning, and subquery decorrelation.
* **9.2 Cost-Based Optimization (CBO):** Histograms, NDV/cardinality estimation, dynamic-programming join reordering, and why misestimated cardinalities cause catastrophic OOMs.

---

## Phase 4: Distributed Query Execution & Real-Time Infra *(Senior Level)*

### 10. Distributed Systems Foundations
* **10.1 Sharding, Consistent Hashing & Hot Partitions:** Range vs. hash partitioning, virtual nodes, rebalancing data without melting the cluster, and hot-key bottlenecks.
* **10.2 Consensus & Metadata Coordination:** How Raft/Paxos, distributed locks, leases, and fencing tokens prevent split-brain corruption in distributed catalogs and coordinators.

### 11. Distributed Query Execution (MPP & Lakehouse Engines)
* **11.1 MPP Topologies & Stage Scheduling:** Coordinator vs. worker nodes (Trino, ClickHouse, Spark), DAG splitting into stages, and pipeline vs. task parallelism.
* **11.2 The Distributed Shuffle & Fault-Tolerant Exchanges:** Repartitioning petabytes across the network by join/group key, exchange buffer backpressure, and batch vs. pipelined recovery on node death.
* **11.3 Data Skew, Adaptive Execution & Stragglers:** Detecting and fixing skewed keys (e.g., `AAPL` having 1,000x more ticks than a small-cap stock) via salting, runtime Adaptive Query Execution (AQE), and speculative retries.

### 12. Real-Time Streaming & Hybrid OLAP
* **12.1 Distributed Logs & Ingestion Backpressure:** Kafka/Redpanda partitioning, consumer group rebalances, at-least-once vs. exactly-once semantics, and absorbing market-open traffic bursts.
* **12.2 Streaming Analytics: Event Time & Watermarks:** Processing out-of-order events, tumbling/sliding windows, watermarks, and late-arriving data trade-offs.
* **12.3 Real-Time OLAP & Hybrid Architectures:** Serving sub-second queries over fresh streams joined with cold historical Parquet lakes (ClickHouse, Apache Pinot, Druid).

---

## Phase 5: Production Rigor, Landmark Papers & Mock Interviews *(Senior → Staff-Ready)*

### 13. Performance Engineering & Benchmarking *(Walk through sample artifacts)*
* **13.1 Systematic Profiling & Flame Graphs:** Reading CPU, allocation, and off-CPU/lock flame graphs (`async-profiler`, JFR, `perf`) to separate real bottlenecks from noise.
* **13.2 Benchmarking Discipline & Hardware Limits:** Evaluating systems via TPC-H/ClickBench, avoiding cold-cache/JIT-warmup traps, and calculating theoretical hardware ceilings (RAM/NVMe/NIC saturation).

### 14. Multi-Tenancy, Governance & Observability *(Walk through configs & traces)*
* **14.1 Query Admission Control & Resource Isolation:** Memory pools, concurrency queues, per-query spill quotas, and protecting the cluster from rogue ad-hoc researcher queries.
* **14.2 Distributed Query Tracing & Correctness Debugging:** Reading distributed execution plans to find tail-latency stragglers and catching silent data-corruption or floating-point aggregation bugs.

### 15. Landmark Paper Breakdowns & Staff Mock Interviews *(Synthesis Phase)*
* **15.1 Paper Breakdown — Google Dremel:** How nested columnar shredding works and why it spawned Parquet and modern cloud warehouses.
* **15.2 Paper Breakdown — Snowflake Elastic Data Warehouse:** Separating stateless compute from S3 storage, metadata services, and local SSD caching.
* **15.3 Paper Breakdown — Databricks Photon:** Why a JVM-based engine hit a wall and how C++ vectorized execution unlocked order-of-magnitude speedups.
* **15.4 Paper Breakdown — Presto/Trino & DuckDB/Velox:** Contrasting distributed MPP federation over data lakes with ultra-fast single-node vectorized execution.
* **15.5 Capstone Mock Design 1 (Batch/Research):** *"Design a petabyte-scale historical market-tick storage & SQL/Python query platform for 200 quant researchers running backtests and ASOF joins."*
* **15.6 Capstone Mock Design 2 (Real-Time/Hybrid):** *"Design a sub-second real-time trading analytics system that ingests 5M events/sec, handles late data and hot-key skew, and joins against 5 years of historical Parquet."*
