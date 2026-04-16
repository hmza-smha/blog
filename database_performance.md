
[Back](./README.md)

# 📊 Database Performance Techniques Cheat Sheet

| Technique             | Category    | What It Does                         | When to Use                        | What It Improves         | Trade-offs                      |
| --------------------- | ----------- | ------------------------------------ | ---------------------------------- | ------------------------ | ------------------------------- |
| **Indexing**          | Data access | Adds data structures for fast lookup | Frequent filtering, joins, sorting | Query speed              | Slower writes, storage overhead |
| **Composite Index**   | Data access | Index on multiple columns            | Multi-column filters               | Query efficiency         | Needs correct column order      |
| **Covering Index**    | Data access | Index contains all needed columns    | Avoid table lookup                 | Faster reads             | Larger index size               |
| **Full-text Index**   | Search      | Enables text search                  | Searching text fields              | Search speed             | Limited use cases               |
| **Columnstore Index** | Analytics   | Stores data column-wise              | Aggregations, OLAP                 | Scan & aggregation speed | Slower writes                   |

---

| Technique         | Category  | What It Does                       | When to Use                  | What It Improves         | Trade-offs       |
| ----------------- | --------- | ---------------------------------- | ---------------------------- | ------------------------ | ---------------- |
| **Partitioning**  | Data size | Splits table into chunks           | Large tables, date filtering | Reduces scanned data     | Added complexity |
| **Sharding**      | Scaling   | Splits DB across servers           | Massive data or writes       | Read + write scalability | Complex joins    |
| **Read Replicas** | Scaling   | Copies DB for reads                | High read traffic            | Read throughput          | Replication lag  |
| **Clustering**    | Scaling   | Multiple DB nodes working together | High availability + scale    | Fault tolerance, scaling | Complex setup    |

---

| Technique                       | Category      | What It Does             | When to Use        | What It Improves   | Trade-offs         |
| ------------------------------- | ------------- | ------------------------ | ------------------ | ------------------ | ------------------ |
| **Caching (Redis / Memcached)** | Avoid DB hits | Stores results in memory | Repeated queries   | Massive speed gain | Cache invalidation |
| **Materialized Views**          | Precompute    | Stores query results     | Heavy aggregations | Instant reads      | Needs refresh      |
| **Denormalization**             | Schema design | Reduces joins            | Read-heavy systems | Faster queries     | Data duplication   |

---

| Technique                          | Category        | What It Does             | When to Use      | What It Improves    | Trade-offs         |
| ---------------------------------- | --------------- | ------------------------ | ---------------- | ------------------- | ------------------ |
| **Connection Pooling (PgBouncer)** | Connection mgmt | Reuses DB connections    | High concurrency | Throughput, latency | Pool tuning needed |
| **Query Routing (ProxySQL)**       | Traffic mgmt    | Routes reads/writes      | Using replicas   | Load balancing      | Extra layer        |
| **Parallel Execution**             | Execution       | Runs queries in parallel | Large scans      | Faster processing   | CPU usage          |

---

| Technique            | Category       | What It Does        | When to Use     | What It Improves  | Trade-offs              |
| -------------------- | -------------- | ------------------- | --------------- | ----------------- | ----------------------- |
| **Compression**      | Storage        | Reduces data size   | Large datasets  | I/O speed         | CPU overhead            |
| **Archiving**        | Data lifecycle | Moves old data out  | Historical data | Smaller active DB | Access complexity       |
| **Batch Processing** | Workload       | Processes in chunks | Heavy writes    | Stability         | Latency (not real-time) |

---

| Technique                         | Category    | What It Does           | When to Use          | What It Improves    | Trade-offs      |
| --------------------------------- | ----------- | ---------------------- | -------------------- | ------------------- | --------------- |
| **Search Engine (Elasticsearch)** | Specialized | External search system | Full-text, filtering | Search performance  | Sync complexity |
| **In-Memory DB (Redis)**          | Storage     | Keeps data in RAM      | Ultra-fast access    | Latency             | Cost (RAM)      |
| **Hardware Scaling**              | Infra       | Better CPU/RAM/SSD     | Resource bottlenecks | Overall performance | Cost            |

---

# 🧭 Quick Decision Table

| Problem            | Best Techniques         |
| ------------------ | ----------------------- |
| Slow queries       | Indexing, columnstore   |
| Too many reads     | Replicas, caching       |
| Large tables       | Partitioning, archiving |
| Write bottlenecks  | Sharding, batching      |
| Heavy aggregations | Materialized views      |
| Full-text search   | Elasticsearch           |
| High concurrency   | Connection pooling      |
| Disk I/O slow      | Compression, SSD        |

---

# ✅ Key Takeaway

* **Make queries efficient → Indexing**
* **Reduce data → Partitioning**
* **Handle load → Replicas**
* **Scale system → Sharding**
* **Avoid work → Caching**

---


Here’s a **step-by-step decision flowchart** you can follow whenever you face a database performance issue. Think of it as a guided path from simplest fixes → advanced scaling.

---

# 🧭 Database Performance Decision Flowchart

```
START
  ↓
Is the query slow?
  ↓
YES → Check execution plan (EXPLAIN ANALYZE)
  ↓
Is it scanning too many rows?
  ↓
YES → Add / fix indexes
  ↓
Still slow?
  ↓
YES → Reduce data (filters, LIMIT, pagination)
  ↓
Still slow?
  ↓
YES → Optimize joins / remove unnecessary work
  ↓
Still slow?
  ↓
YES → Use columnstore / materialized view (for aggregation)
  ↓
Still slow?
  ↓
YES → Cache results (Redis)
  ↓
Still slow?
  ↓
YES → Partition large tables
  ↓
Still slow?
  ↓
YES → Consider sharding
  ↓
END
```

---

# 🔁 Parallel Path: System Load Problem

```
START
  ↓
Is database CPU / load high?
  ↓
YES → Are queries already optimized?
  ↓
NO → Fix queries first
  ↓
YES
  ↓
Is workload read-heavy?
  ↓
YES → Add read replicas
  ↓
Still overloaded?
  ↓
YES → Add caching (Redis)
  ↓
Still overloaded?
  ↓
YES → Scale hardware (CPU/RAM/SSD)
  ↓
Still overloaded?
  ↓
YES → Shard database
  ↓
END
```

---

# 📊 Data Size Problem

```
Is table very large?
  ↓
YES
  ↓
Do queries filter by a column (e.g., date)?
  ↓
YES → Use partitioning
  ↓
NO → Consider sharding
```

---

# 🔎 Search-Specific Problem

```
Is query doing text search or complex filtering?
  ↓
YES → Use Elasticsearch
```

---

# ⚡ Simplified Mental Flow (Interview Version)

1. **Check execution plan**
2. **Fix indexes**
3. **Reduce scanned data**
4. **Optimize joins**
5. **Precompute (materialized views / columnstore)**
6. **Cache results**
7. **Scale reads (replicas)**
8. **Split data (partitioning)**
9. **Scale writes (sharding)**

---

# 🧠 Key Insight (Most Important Rule)

👉 **Never jump to sharding or replicas first**
Always go in this order:

**Query → Index → Data → Cache → Scale**

---

# ✅ Real-World Example Walkthrough

**Problem:** Dashboard query is slow

Flow:

1. EXPLAIN → full table scan
2. Add index ✅
3. Still slow → aggregation heavy
4. Add materialized view ✅
5. Traffic grows → DB CPU high
6. Add read replicas ✅
7. Still heavy → add Redis caching ✅

---


Great question — this is exactly where people level up in performance tuning. Execution plans can look overwhelming, but once you understand the **operators**, everything becomes much clearer.

Here’s a **comprehensive table of the most important execution plan operators**, what they mean, when they appear, and whether they’re good or bad.

---

# 📊 Execution Plan Operators Explained

## 🔍 Access Methods (How data is read)

| Operator                     | What It Means                              | When It Happens                         | Good or Bad         | Notes                                  |
| ---------------------------- | ------------------------------------------ | --------------------------------------- | ------------------- | -------------------------------------- |
| **Table Scan**               | Reads entire table                         | No index OR optimizer chooses full scan | ❌ Usually bad       | OK only for very small tables          |
| **Index Scan**               | Reads entire index                         | Index exists but not selective enough   | ⚠️ Medium           | Better than table scan but still heavy |
| **Index Seek**               | Directly jumps to matching rows            | Proper index + selective filter         | ✅ Best              | What you usually want                  |
| **Clustered Index Scan**     | Scans full table (since clustered = table) | No good filter on clustered key         | ❌ Bad               | Same as table scan logically           |
| **Clustered Index Seek**     | Seeks using primary key                    | Query filters on clustered key          | ✅ Excellent         | Fastest access                         |
| **Non-Clustered Index Scan** | Scans secondary index                      | Index exists but not selective          | ⚠️ Medium           | Might still scan many rows             |
| **Non-Clustered Index Seek** | Uses secondary index efficiently           | Filter matches index                    | ✅ Good              | Often followed by Key Lookup           |
| **Key Lookup (RID Lookup)**  | Fetches missing columns from table         | Index doesn't cover query               | ⚠️ Can be expensive | Fix with covering index                |

---

## 🔗 Join Operators (How tables are combined)

| Operator                   | What It Means                              | When It Happens               | Good or Bad           | Notes                        |
| -------------------------- | ------------------------------------------ | ----------------------------- | --------------------- | ---------------------------- |
| **Nested Loops Join**      | Loops through one table and probes another | Small dataset OR indexed join | ✅ Good for small data | Becomes slow with large data |
| **Hash Match (Hash Join)** | Builds hash table in memory                | Large datasets, no indexes    | ⚠️ Medium             | Memory-heavy but scalable    |
| **Merge Join**             | Joins sorted datasets                      | Both inputs sorted/indexed    | ✅ Very fast           | Requires sorted inputs       |

---

## 📊 Aggregation Operators

| Operator             | What It Means                | When It Happens      | Good or Bad | Notes            |
| -------------------- | ---------------------------- | -------------------- | ----------- | ---------------- |
| **Stream Aggregate** | Aggregates sorted data       | Input already sorted | ✅ Efficient | Preferred        |
| **Hash Aggregate**   | Uses hash table for grouping | Unsorted large data  | ⚠️ Medium   | Uses more memory |

---

## 🔄 Sorting & Filtering

| Operator   | What It Means           | When It Happens                   | Good or Bad | Notes                    |
| ---------- | ----------------------- | --------------------------------- | ----------- | ------------------------ |
| **Sort**   | Sorts result set        | ORDER BY / GROUP BY without index | ❌ Expensive | Avoid with proper index  |
| **Filter** | Applies WHERE condition | After data retrieval              | ⚠️ Neutral  | Better to filter earlier |

---

## 🔁 Data Movement / Intermediate

| Operator                      | What It Means           | When It Happens            | Good or Bad         | Notes                   |
| ----------------------------- | ----------------------- | -------------------------- | ------------------- | ----------------------- |
| **Compute Scalar**            | Computes expressions    | Calculations, functions    | ✅ Cheap             | Usually harmless        |
| **Concatenation**             | Combines result sets    | UNION ALL                  | ✅ Fine              | Lightweight             |
| **Spool (Table/Index Spool)** | Temporary storage       | Reuse intermediate results | ⚠️ Can be expensive | Indicates repeated work |
| **Parallelism**               | Splits work across CPUs | Large queries              | ✅ Good              | Watch for overhead      |

---

## ⚠️ Warning Signs (Red Flags)

| Operator                | Why It's Bad         |
| ----------------------- | -------------------- |
| Table Scan              | Reads entire table   |
| Clustered Index Scan    | Same as table scan   |
| Key Lookup (many times) | Causes repeated I/O  |
| Sort (large data)       | Expensive memory/CPU |
| Hash Match (huge data)  | Memory spill risk    |
| Spool                   | Hidden repeated work |

---

# 🧭 Quick Interpretation Cheat Sheet

## If you see:

### 🔴 Table Scan

→ Missing index → **Add index**

---

### 🟡 Index Scan

→ Index exists but inefficient
→ **Improve selectivity / composite index**

---

### 🟢 Index Seek

→ ✅ Good — leave it

---

### 🔴 Key Lookup (many times)

→ **Create covering index**

---

### 🔴 Nested Loop + Large Rows

→ Bad choice →
→ Try **Hash Join or better indexing**

---

### 🔴 Sort Operator

→ Add index for ORDER BY

---

### 🔴 Hash Match with spills

→ Memory issue →
→ Add index or reduce dataset

---

# 🧠 How to Read a Plan (Simple Method)

1. Start from **right to left** (data flow)
2. Look for:

   * Biggest cost operators
   * Scans vs seeks
3. Ask:

   * Can I reduce rows earlier?
   * Can I avoid this operator?

---

# ⚡ Example Mental Mapping

| You See          | You Think                   |
| ---------------- | --------------------------- |
| Table Scan       | “I need an index”           |
| Key Lookup       | “I need covering index”     |
| Sort             | “I need index for ORDER BY” |
| Hash Join        | “No index on join”          |
| Nested Loop slow | “Too many rows”             |

---

# ✅ Final Insight

Execution plans answer one question:

👉 **“How is the database actually executing my query?”**

Once you map operators to problems:

* Scans → indexing issue
* Joins → data size/index issue
* Sorts → missing index
* Lookups → incomplete index

---

## Questions

Great — this is exactly how you turn knowledge into **interview-ready skill**. I’ll give you a **well-structured question bank** covering everything we discussed, from basics → advanced → real scenarios.

---

# 🧠 1. Fundamentals (Warm-up Questions)

### 🔹 Basic Concepts

* What is an execution plan and why is it important?
* What is the difference between **Index Seek** and **Index Scan**?
* What is a **Table Scan**, and when is it acceptable?
* What is a **clustered vs non-clustered index**?
* What is a **covering index**?

---

### 🔹 Query Optimization Basics

* Why is `SELECT *` considered bad practice?
* What happens when you use functions on indexed columns?
* What is the difference between `IN` and `EXISTS`?
* Why are joins sometimes slower than expected?

---

# 🔍 2. Execution Plan Deep Dive

### 🔹 Operators Understanding

* Explain **Nested Loop Join**, **Hash Join**, and **Merge Join**
* When does the optimizer choose a **Hash Match**?
* What is a **Key Lookup** and why is it expensive?
* What is a **Sort operator**, and how can you eliminate it?

---

### 🔹 Scenario-Based

* You see a **Table Scan** in the execution plan — what do you do?
* You see an **Index Scan instead of Seek** — why?
* You see **Key Lookup repeated thousands of times** — how do you fix it?
* You see a **Hash Join spilling to disk** — what does that mean?

---

# 📊 3. Indexing Questions (Very Common)

* How do you decide which columns to index?
* What is a **composite index**, and how does column order matter?
* What is **index selectivity**?
* When should you NOT create an index?
* What is a **covering index vs included columns**?

---

### 🔹 Practical Scenario

> You have a query filtering on 2 columns but only one index exists — what do you do?

---

# 🧱 4. Database-Level Optimization

### 🔹 Core Concepts

* What is **partitioning**, and when should you use it?
* What is **sharding**, and how is it different from partitioning?
* What are **read replicas**, and when do you use them?
* What is **denormalization**, and why would you use it?

---

### 🔹 Comparison Questions

* Partitioning vs Sharding — explain with examples
* Caching vs Replicas — when to use each
* Columnstore vs row-based storage

---

# ⚡ 5. Scaling & Architecture

* How do you scale a database for **read-heavy workloads**?
* How do you scale for **write-heavy workloads**?
* When would you introduce Redis?
* When would you use Elasticsearch instead of a database?

---

### 🔹 Real Scenario

> Your database CPU is high due to many SELECT queries — what steps do you take?

---

# 🧪 6. Real Optimization Scenarios (IMPORTANT)

### Scenario 1

> A query is slow. Execution plan shows **Table Scan**

* What steps do you take?
* What could be the root cause?

---

### Scenario 2

> Query uses index but still slow → shows **Key Lookup**

* What is happening?
* How do you fix it?

---

### Scenario 3

> Query with `ORDER BY` is slow

* What operator appears?
* How do you optimize it?

---

### Scenario 4

> Aggregation query on huge table is slow

* What operators appear?
* What are 3 ways to optimize it?

---

### Scenario 5

> Large table (billions of rows), queries filter by date

* What solution do you apply?
* Why?

---

# 🔁 7. Decision-Making Questions (Senior Level)

* Walk me through how you troubleshoot a slow query step-by-step
* When would you choose:

  * Indexing vs caching?
  * Partitioning vs sharding?
* What’s your **optimization workflow**?

---

### 🔹 Expected Answer Structure:

You should say:

> “First I check execution plan → then indexing → reduce scanned data → optimize joins → consider caching → then scale using replicas or partitioning.”

---

# ⚠️ 8. Trick / Trap Questions

* Is adding an index always good?
* Do replicas make queries faster?
* Can partitioning replace indexing?
* Is sharding always the best scaling solution?
* Why can an index sometimes NOT be used?

---

# 🧠 9. Advanced / Expert Questions

* What causes a **bad execution plan**?
* What is **parameter sniffing**?
* What is **cardinality estimation**?
* How does the optimizer choose join types?
* What causes **index fragmentation**?

---

# 🎯 10. Behavioral + Real Experience

* Tell me about a time you optimized a slow query
* What was the biggest performance issue you solved?
* How do you measure performance improvements?

---

# 🧭 11. Whiteboard / System Design Style

* Design a system handling millions of reads per second
* How would you scale a reporting database?
* How would you design a search system?

---

# ✅ How to Practice (Important)

When answering, ALWAYS:

1. Start with **execution plan**
2. Mention **indexing**
3. Mention **data reduction**
4. Then **scaling (replicas, partitioning, sharding)**

---

# 🚀 Pro Interview Tip

A strong answer sounds like:

> “I checked the execution plan and saw a clustered index scan. I added a composite covering index, which converted it into an index seek and eliminated key lookups. Then I reduced load further by caching frequent queries.”

---


[Back](./README.md)
