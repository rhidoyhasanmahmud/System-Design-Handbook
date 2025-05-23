# What Is Database Indexing?

A database index is like a table of contents for your data. It helps the database find rows faster, just like how a book index helps you find topics quickly without reading every page.

**Why Use Indexes?**

Without an index:

- A query must scan every row in the table → Full Table Scan
- Slow for large tables

With an index:

- DB engine can jump to the data it needs → Fast Lookup

**Real-Life Analogy**

Imagine a library:

| Book Search Type         | Time Taken |
|--------------------------|------------|
| 🔍 Search every shelf     | 🕒 Slow     |
| 📘 Use book index (TOC)   | ⚡ Fast     |

Same with a database — indexes speed up searching.

**Types of Indexes**

| Type                 | Description                                  | Example Use Case                            |
|----------------------|----------------------------------------------|---------------------------------------------|
| **Primary Index**     | Automatically created on Primary Key         | `id` in `users(id PRIMARY KEY)`             |
| **Secondary Index**   | Manually created on non-primary columns      | `email`, `name`, `status`                   |
| **Unique Index**      | Ensures no duplicate values                  | `email` must be unique                      |
| **Composite Index**   | Index over multiple columns                  | `INDEX (first_name, last_name)`             |
| **Full-Text Index**   | Optimized for text search                    | `MATCH(content) AGAINST('keyword')`         |
| **Hash Index**        | Uses hash functions instead of tree structure| Fast equality lookups (`=`), not ranges     |
| **Clustered Index**   | Data rows are sorted physically by this index| InnoDB: PK = clustered index                |
| **Non-Clustered Index**| Index stored separately from table          | Most secondary indexes                      |

**How Indexes Work (Flow)**

```text
SELECT * FROM users WHERE name = 'Zara';

1. Query sent to DB
2. DB checks if 'name' is indexed
3. If indexed → uses B-Tree to find value
4. Finds pointer to row
5. Fetches row from table page
```
Without index → DB would read every row.

**When NOT to Use Indexes**

| Situation                                | Why Not Index?                          |
|------------------------------------------|------------------------------------------|
| 🟦 Columns frequently updated             | Index needs constant maintenance         |
| 📏 Small tables                          | Full scan may be faster                  |
| 📊 Low-cardinality columns (e.g., gender)| Index not selective enough               |
| 🗒️ Heavy insert load                      | Index slows down insert speed            |

**Best Practices**
1. Index columns used in WHERE, JOIN, ORDER BY
2. Use composite indexes wisely (order matters!)
3. Don’t over-index — hurts write performance
4. Monitor with EXPLAIN / Query Plan

---

# What Is a Query Planner and Optimizer?

When you write an SQL query, the database engine doesn’t just run it blindly. It does 3 major steps:

1. Parsing: Checks if your SQL is valid (e.g., grammar and table/column existence).

2. Planning (Query Planner): Generates multiple strategies (called execution plans) to fetch the data.

3. Optimizing (Query Optimizer): Chooses the best and most efficient plan based on data size, indexes, and statistics.

**Why Is It Important?**

The optimizer can turn a slow 10-second query into a fast 10-millisecond one, just by choosing a smarter way to access
the data.

**Using ```EXPLAIN```**

```sql
CREATE TABLE employees
(
    id            SERIAL PRIMARY KEY,
    name          VARCHAR,
    department_id INT,
    salary        INT
);
```

Now, let’s run a query:

```sql
SELECT *
FROM employees
WHERE department_id = 2;
```

Step 1: Use EXPLAIN

```sql
EXPLAIN
SELECT *
FROM employees
WHERE department_id = 2;
```

Sample Output (PostgreSQL-style):

```pgsql
Seq Scan on employees  (cost=0.00..35.50 rows=5 width=40)
  Filter: (department_id = 2)
```

**Interpretation**

- Seq Scan (Sequential Scan): The planner chooses to scan every row in the table — not efficient if table is large.

- Cost=0.00..35.50: Estimate of how expensive this operation is. Lower is better.

- Rows=5: Estimated number of rows matching the filter.

- width=40: Average row size (in bytes).

**Better Plan With Index**

Let’s add an index and recheck:

```sql
CREATE INDEX idx_dept_id ON employees (department_id);
EXPLAIN
SELECT *
FROM employees
WHERE department_id = 2;
```

Now the plan may look like:

```pgsql
Index Scan using idx_dept_id on employees  (cost=0.12..8.50 rows=5 width=40)
  Index Cond: (department_id = 2)
```

| Term       | Meaning                               |
|------------|---------------------------------------|
| Seq Scan   | Full table scan                       |
| Index Scan | Uses index to find rows               |
| Cost       | Estimated effort for the operation    |
| Rows       | Estimated result row count            |
| Filter     | Rows filtered after access            |
| Index Cond | Condition applied during index lookup |

---

# Bitmap Index Scan vs Index Scan vs Table Scan

**1. Table Scan (Sequential Scan)**

Reads every row in the table, no index used.

When Used:

- No index exists on the column.
- The table is small.
- The filter matches a large portion of the table.

Think Like: “Just loop through everything and check condition.”

Downside: Slow on large tables.

**2. Index Scan**

Traverses the B-tree index to find matching rows. Then, for each match, fetches the actual row from the table (random
I/O).

When Used:

* Index exists.
* Query returns few matching rows (high selectivity).
* Good for range queries and unique lookups.

Think Like: “Find keys in the index, then jump to table rows one by one.”

Downside: Can be slower if many rows are returned due to random I/O.

**3. Bitmap Index Scan**

Uses index to collect all matching row locations first (bitmap in memory), then fetches them in bulk (sorted order) from
the table.

When Used:

* Index exists.
* Query returns many rows (medium selectivity).
* Often used with multiple conditions or OR.

Think Like: “Mark all the matching rows first, then read them all at once.”

Advantage: More I/O efficient than Index Scan for large result sets.

**Summary Table**

| Scan Type         | Uses Index? | Best For                       | I/O Pattern    | Speed (general)     |
|-------------------|-------------|--------------------------------|----------------|---------------------|
| Seq Scan          | ❌           | Small tables / many matches    | Sequential     | 🟡 Medium           |
| Index Scan        | ✅           | Few matching rows              | Random         | 🟢 Fast (few rows)  |
| Bitmap Index Scan | ✅✅          | Many rows / complex conditions | Bulk (batched) | 🟢 Fast (many rows) |

---

# Key vs Non-Key Column Indexing

**1. Key Column Indexing**

Indexes are built on columns that uniquely identify rows (like Primary Keys or Unique Keys).

```sql
CREATE TABLE users
(
    id    INT PRIMARY KEY,
    email VARCHAR UNIQUE,
    name  VARCHAR
);
```

* id and email are key columns.
* They are automatically indexed in most RDBMS (e.g., PostgreSQL, MySQL).

**Why Important:**

* Ensures uniqueness.
* Supports fast lookup via WHERE, JOIN, UPDATE, etc.

**Index Use Case:**

```sql
SELECT *
FROM users
WHERE id = 10;
```

→ Fast lookup using index on id.

**2. Non-Key Column Indexing**

You manually create indexes on columns that are not unique and not part of the primary/unique keys.

```sql
CREATE INDEX idx_name ON users (name);
```

* name is a non-key column.
* Used to speed up filtering, sorting, or joining.

**Index Use Case:**

```sql 
SELECT *
FROM users
WHERE name = 'Hasan';
```

→ Index idx_name helps find rows faster.

**Comparison Table**

| Feature               | Key Column Index          | Non-Key Column Index     |
|-----------------------|---------------------------|--------------------------|
| Automatically created | ✅ (Primary/Unique keys)   | ❌ (You create manually)  |
| Ensures uniqueness    | ✅                         | ❌                        |
| Ideal for             | Lookups, joins, integrity | Filters, sorts, group by |
| Contains NULL values  | Usually ❌                 | ✅                        |
| Typical Index Type    | B-tree                    | B-tree / Bitmap / etc.   |

**When to Add Index on Non-Key Columns?**

Use when:

* Column is frequently used in WHERE, JOIN, ORDER BY.
* Table is large.
* Query performance is slow without it.

Avoid if:

* Table is small.
* Column is updated frequently (index maintenance overhead).

---

# What is VACUUM?

When you UPDATE or DELETE rows, PostgreSQL doesn't immediately remove the old row — it marks it as dead for MVCC (
Multi-Version Concurrency Control).

VACUUM:

* Removes these dead tuples.
* Frees up space for reuse.
* Prevents table bloat.
* Updates table and index statistics for the planner (in ANALYZE or VACUUM ANALYZE).

**What does VERBOSE do?**

VERBOSE shows detailed output of what VACUUM is doing.

```sql
VACUUM
VERBOSE tablename;
```

Example Output:

```vbnet
INFO:  vacuuming "public.employees"
INFO:  "employees": found 1000 removable, 500 nonremovable row versions in 500 pages
...
```

| Command          | Description                                              |
|------------------|----------------------------------------------------------|
| `VACUUM`         | Reclaims space                                           |
| `VACUUM VERBOSE` | Shows detailed cleanup log                               |
| `VACUUM ANALYZE` | Also updates planner stats                               |
| `VACUUM FULL`    | Shrinks table by rewriting it fully 🚨 (Locks the table) |

**When Should You Run It?**

* Regularly (autovacuum does it by default).
* After massive DELETEs or UPDATEs.
* If you're analyzing query slowdowns or bloat.
* In dev/testing, use VERBOSE to learn behavior.

**Table bloat**

Table bloat in PostgreSQL (or other MVCC-based databases) refers to the wasted space in a table caused by dead or
outdated rows that haven't been physically removed from disk yet.

_Why Does It Happen?_

PostgreSQL uses MVCC (Multi-Version Concurrency Control), which means:

- When you UPDATE a row:

  → It creates a new version of the row and marks the old one as dead.

- When you DELETE a row:

  → The row is not immediately removed, just marked as dead.

Over time, these dead rows pile up, causing:

* Bigger table size on disk
* Slower sequential scans and vacuum
* Increased I/O and performance degradation

---

# Index Scan and Index Only Scan

**1. Index Scan**

- Scans the index to find row locations (CTIDs).
- Then fetches actual rows from the table (heap).

When It’s Used:

- The index helps narrow down rows.
- But the query needs columns not stored in the index (e.g., SELECT *).

Think Like: "Use index to find rows, but still go to table to fetch full data."

Downside: Extra I/O for heap fetch = slower for many rows.

**2. Index Only Scan**

* Reads data directly from the index.
* Skips heap/table fetch completely.

When It’s Used:

* All required columns are in the index.
* The index is "visible" to all (i.e., VACUUM has cleared dead rows).

Think Like: "All needed data is already in the index — no need to touch the table."

Advantage:

* Faster (no table access, less I/O).

**Summary**

| Feature               | Index Scan            | Index Only Scan              |
|-----------------------|-----------------------|------------------------------|
| Reads table (heap)?   | ✅ Yes                 | ❌ No                         |
| Needs extra I/O?      | ✅ Yes (table access)  | ❌ No                         |
| Speed (when possible) | 🟡 Medium             | 🟢 Fast                      |
| Use case              | Partial data in index | All data in index            |
| Requires vacuum?      | ❌ No                  | ✅ Yes (to mark rows visible) |

**Example**

Imagine a Table: users

And you have this index:

```sql
CREATE INDEX idx_email ON users (email);
```

**Scenario 1: Index Scan**

```sql
SELECT name
FROM users
WHERE email = 'b@example.com';
```

**What Happens:**

* PostgreSQL looks in the index (idx_email) and finds that email = 'b@example.com' exists at row ID 2.
* But name is not in the index, so:
* It jumps to the main table (heap) to fetch the full row (id, email, name).
* It returns name = Bob.

Steps:

* Index Lookup
* Heap Fetch
* Return Result

****Scenario 2: Index Only Scan****

```sql
SELECT email
FROM users
WHERE email = 'b@example.com';
```

**What Happens:**

1. PostgreSQL looks in the index.
2. email is already in the index — it’s all the query needs.
3. No heap access needed.
4. It directly returns 'b@example.com'.

**Important:**

This only works if:

* The query only needs columns stored in the index.
* PostgreSQL knows the index row is visible to all transactions (VACUUM helps here).

**Visual Summary**

Index Scan:

```pgsql
[Index] ➡ finds row pointer ➡ [Heap] ➡ fetch row ➡ return data
```

Index Only Scan:

```pgsql
[Index] ➡ return data directly (no heap access)
```

---

# Combining database indexes better performances

Combining database indexes—also known as composite (multi-column) indexes—can significantly improve performance when
used correctly.

**What Is a Composite Index?**

An index on two or more columns, used together.

```sql
CREATE INDEX idx_users_email_name ON users (email, name);
```

Now the index includes both email and name — in that order.

**How It Works**

Composite indexes are left-to-right searchable:

| Query                              | Uses Index `idx_users_email_name`? |
|------------------------------------|------------------------------------|
| `WHERE email = 'x'`                | ✅ Yes (prefix match)               |
| `WHERE email = 'x' AND name = 'y'` | ✅ Yes (full match)                 |
| `WHERE name = 'y'`                 | ❌ No (starts from second column)   |

So:

* Composite index (email, name)
* Can help: queries with email or email + name
* Cannot help: name alone

**Why Combine Indexes?**

* To avoid multiple index scans and joins
* To speed up filtering and sorting on multiple columns
* To help with composite WHERE clauses (like: WHERE status = 'active' AND created_at > NOW() - INTERVAL '7 days')

**Best Practices**

* Create composite indexes for columns frequently queried together.
* Always match index order to your query pattern.
* Use EXPLAIN ANALYZE to confirm index usage.
* Don’t over-index — monitor with tools like pg_stat_user_indexes.

## Caveats

| Concern            | Explanation                                      |
|--------------------|--------------------------------------------------|
| Index size         | Larger composite indexes take more disk space    |
| Write overhead     | More indexes = more cost on INSERT/UPDATE/DELETE |
| Wrong column order | Useless if query doesn't match the index order   |
| Duplicated indexes | Avoid overlapping indexes unnecessarily          |

---

# How Database Optimizers Decide to Use Indexes?

The query optimizer is a smart component of a database engine (like PostgreSQL, MySQL, SQL Server) that:

- Evaluates multiple execution plans
- Estimates cost of each (I/O, CPU, memory)
- Picks the cheapest and fastest one

**How Does It Decide to Use an Index?**

1. Selectivity

* Definition: How many rows match the condition?
* Rule: The fewer rows matched, the more likely the index will be used.
* Index is preferred when the query filters on rare values.

```sql
SELECT *
FROM users
WHERE email = 'x@example.com';
-- Uses index if email is unique or rare
```

2. Statistics

* The optimizer relies on table and column statistics: row count, distinct values, null count, histograms.
* Kept up to date by:

```sql
ANALYZE; -- or AUTOVACUUM
```

If stats are outdated → optimizer may make bad choices.

3. Query Shape

* Index is useful for:

    * WHERE filters
    * JOIN conditions
    * ORDER BY
    * GROUP BY
    * LIMIT

```sql
SELECT *
FROM orders
WHERE customer_id = 5
ORDER BY created_at DESC LIMIT 10;
-- Composite index on (customer_id, created_at DESC) is perfect
```

4. Index Coverage

* If the index includes all needed columns, optimizer may choose:
    * Index Only Scan (no heap access = faster)
* Otherwise, needs extra heap fetch → might choose Seq Scan

5. Cost Model

The optimizer calculates cost using:

| Factor        | Impact                      |
|---------------|-----------------------------|
| Disk I/O      | High                        |
| Memory        | Medium                      |
| CPU (filters) | Low                         |
| Row estimates | Crucial for decision making |

6. Index Type & Ordering

* B-Tree: Good for range lookups and equality
* Bitmap: Good for OR and multiple filters
* Hash: Equality only
* Optimizer checks if index matches filter pattern and sort order

7. Available Resources

* If memory is low or disk I/O is expensive → index may help more.
* Optimizer adapts based on available settings (like work_mem, random_page_cost).

**When Optimizer Might NOT Use Index**

* Table is very small → Seq Scan is faster
* Index is on a low-selectivity column (e.g., gender = 'M')
* Stats are missing or outdated
* Function or expression is not indexable

```sql
WHERE LOWER(email) = 'abc@example.com' -- unless index on LOWER(email)
```

**Best Practices**

* Keep stats updated: ANALYZE or autovacuum
* Use EXPLAIN (ANALYZE) to confirm index use
* Create composite indexes when filtering by multiple columns
* Avoid writing queries that hide indexed columns in functions or casts

Let’s walk through a real example showing how the database optimizer decides whether to use an index using EXPLAIN
ANALYZE.

**Step 1: Create a Table**

```sql
CREATE TABLE users
(
    id      SERIAL PRIMARY KEY,
    email   TEXT,
    age     INT,
    country TEXT
);
```

Now insert some sample data:

```sql
INSERT INTO users (email, age, country)
SELECT 'user' || i || '@example.com',
       (random() * 100)::int, CASE WHEN i % 2 = 0 THEN 'Malaysia' ELSE 'Bangladesh' END
FROM generate_series(1, 100000) AS i;
```

**Step 2: Add Index**

```sql
CREATE INDEX idx_users_country ON users (country);
```

**Step 3: Query with and without Selectivity**

**A. Low Selectivity Query (many matches)**

```sql
EXPLAIN
ANALYZE
SELECT *
FROM users
WHERE country = 'Malaysia';
```

Expected: PostgreSQL may use Sequential Scan because many rows match.

Sample Output:

```pgsql
Seq Scan on users  (cost=0.00..1520.00 rows=50000 width=40)
  Filter: (country = 'Malaysia')
```

**B. High Selectivity Query (few matches)**

```sql
-- Add rare country
INSERT INTO users (email, age, country)
VALUES ('admin@rare.com', 45, 'Canada');

-- Query
EXPLAIN
ANALYZE
SELECT *
FROM users
WHERE country = 'Canada';
```

Expected: Index should be used because only 1 row matches.

Sample Output:

```sql
Index
Scan using idx_users_country on users
  Index Cond: (country = 'Canada')
```

✅ Optimizer chooses Index Scan due to high selectivity.

**Why This Matters**

| Query                        | Match Size | Optimizer Chooses |
|------------------------------|------------|-------------------|
| `country = 'Malaysia'`       | ~50,000    | Sequential Scan   |
| `country = 'Canada'` (1 row) | 1          | Index Scan        |

---

# Create Index Concurrently - Avoid Blocking Production Database Writes

Creating indexes on large tables in a production database can block reads/writes and cause downtime. To avoid blocking
writes, PostgreSQL provides the powerful option:

**CREATE INDEX CONCURRENTLY**

```sql
CREATE INDEX CONCURRENTLY index_name ON table_name (column_name);
```

What It Does:

* Does NOT lock the table for writes.
* Allows inserts/updates/deletes while the index is being created.
* Ideal for live production environments.

## Caveats & Considerations

| Point                                   | Details                                                        |
|-----------------------------------------|----------------------------------------------------------------|
| Slower than normal `CREATE INDEX`       | Because it builds index without locking and in multiple phases |
| Cannot be used in a transaction block   | ❌ Must run outside `BEGIN ... COMMIT`                          |
| Requires 2 scans                        | One for index build, another for catching concurrent changes   |
| Index is only usable **after complete** | It won’t be used in queries until fully built                  |

Example of Wrong Usage:

```sql
BEGIN;
CREATE INDEX CONCURRENTLY idx_email ON users(email); -- ❌ Error: not allowed inside transaction block
COMMIT;
```

Correct Usage Example:

```sql
-- Safe to run in production
CREATE INDEX CONCURRENTLY idx_email ON users(email);
```

**When Should You Use It?**

Use CONCURRENTLY when:

* You're on a live production DB
* The table is large
* You want zero downtime
* You can tolerate slightly longer index creation time

**Cleanup (DROP INDEX CONCURRENTLY)**

```sql
DROP INDEX CONCURRENTLY idx_email;
```

---

# Bloom Filter

A Bloom Filter is a space-efficient, probabilistic data structure used to test whether an element is possibly in a set
or definitely not.

* Think of it like a super-fast “membership checker.”
* It can say:

    * "Maybe it’s there"
    * "Definitely not there"
* It can never produce false negatives, but may produce false positives.

**How It Works**

1. You have a bit array (e.g., size 1,000 bits), all initially set to 0.
2. You use multiple hash functions (e.g., 3 different hash algorithms).
3. For each item inserted:
    * Apply each hash → get positions in the bit array
    * Set those bits to 1

**Checking for an item:**

* Apply the same hashes
* Check the bit positions
* If any are 0 → definitely not in the set
* If all are 1 → maybe in the set

**Visual Example**

```bash
Bit Array: [0, 1, 0, 1, 0, 0, 1, ...] ← bits set by hash("apple") and hash("banana")
```

Check “mango”:

* Hashes point to bits that are all 1 → maybe
* Check “orange”:
    * One bit is 0 → definitely not

**Pros**

| Benefit            | Description                   |
|--------------------|-------------------------------|
| Space-efficient    | Uses very little memory       |
| Fast lookup        | Constant time (O(1))          |
| No false negatives | If says "no", it’s guaranteed |

**Cons**

| Limitation             | Description                              |
|------------------------|------------------------------------------|
| False positives        | It might say "maybe" even if not present |
| No deletion (standard) | Deleting items isn't straightforward     |
| Fixed size             | Must size appropriately ahead of time    |

**Use Cases**

* Caches (e.g., check if key might be in cache before expensive lookup)
* Databases (e.g., PostgreSQL’s Bloom Index extension for multicolumn searches)
* Web crawlers (avoid revisiting URLs)
* Distributed systems (quick existence check)

**Scenario: Block Spam Emails**

You want to build a system that quickly checks if an email address is blacklisted (spam). A Bloom Filter helps you
quickly reject known spam before doing expensive checks.

> WIP 



