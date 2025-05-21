# How Tables and Indexes Are Stored on Disk

1. Tables (Heap Files or Organized Files)

A table in a database is stored as a collection of rows (records), and they are typically stored on disk in pages (blocks).

Each row = a record
Each page = a chunk of data (like 8KB)

Types of Table Storage:
- Heap file: Rows are stored in no particular order.
- Clustered/Sorted file: Rows are stored based on the order of one column (e.g., by id).

Example:

A table:

| ID | Name  | Age |
|----|-------|-----|
| 1  | Alice | 23  |
| 2  | Bob   | 25  |
| 3  | Zara  | 21  |

These records will be stored in data pages, like:

```pgsql
Page 1:
+---------------------+
| Record: ID=1        |
| Record: ID=2        |
| Record: ID=3        |
+---------------------+
```

Each page has:

- Header (metadata)
- Multiple rows
- Free space for new inserts (if available)

2. Indexes

An index is like a table of contents in a book. It helps you find data fast without scanning the whole table.

Most common: B-Tree Index

Suppose you create this index:

```sql
CREATE INDEX idx_name ON users(name);
```
This creates a tree structure like:

```css
        [K]
      /     \
   [A-C]   [L-Z]
```

So if you're looking for Zara, DB directly goes to [L-Z] node instead of reading all users.

✅ Index stores column value + pointer to the actual row.

# How Tables and Indexes Are Queried

Without Index:

```sql
SELECT * FROM users WHERE name = 'Zara';
```

🔁 DB scans the whole table (called Full Table Scan)
⛔ Slow if table has millions of rows.

With Index:

```sql
SELECT * FROM users WHERE name = 'Zara';
```

✅ DB uses the index tree, finds the pointer, and jumps directly to the data row.
💡 Much faster – no need to scan entire table.

---

## What is a Heap File?

A Heap File is the simplest way to store data in a database table:

- Rows are stored in the order they are inserted.
- No sorting, no structure — just a pile of rows.

Characteristics:

- Fast insert
- No order
- Slower search (needs to check every row unless indexed)

Real-life Analogy:

Like throwing all your documents in a drawer - to find one, you may need to go through all of them.

Example:

Insert 3 rows:

```sql
INSERT INTO users VALUES (3, 'Zara', 21);
INSERT INTO users VALUES (1, 'Alice', 23);
INSERT INTO users VALUES (2, 'Bob', 25);
```
In a heap file, rows might be stored like:

```yaml
Page 1:
- Row 1: Zara
- Row 2: Alice
- Row 3: Bob
```
They’re not sorted by ID or name.

## What is an Organized (Sorted/Clustered) File?

An Organized File stores data sorted by a key column (like id, name, etc.).
This structure is also called a clustered index when used in many RDBMS (e.g., MySQL InnoDB).

Characteristics:
- Rows stored in order based on key
- Faster range queries (e.g., WHERE id BETWEEN 10 AND 20)
- Slower insert/update (needs to maintain order)

Real-life Analogy:

Like storing your documents in alphabetically sorted folders — easy to find and range-search.

Example: Same rows, but stored by ID:

```yaml
Page 1:
- Row 1: Alice (ID 1)
- Row 2: Bob (ID 2)
- Row 3: Zara (ID 3)
```

Now, searching WHERE id = 2 or WHERE id BETWEEN 1 AND 2 is faster.

## Key Differences

| Feature           | Heap File                          | Organized File (Clustered)               |
|-------------------|------------------------------------|------------------------------------------|
| Storage Order     | Random (insertion order)           | Sorted by one column (e.g., ID)          |
| Insert Speed      | Fast                               | Slower (due to sorting)                  |
| Read/Search Speed | Slow (unless indexed)              | Fast for range queries                   |
| Use Case          | OLTP apps (lots of writes)         | Reporting apps (lots of reads)           |

## When to Use What?

- Heap File: Good when you mostly insert data and don’t care about order.
E.g., logging systems, message queues, temporary staging tables.
- Organized File: Best when you frequently read/search in sorted order.
E.g., analytics dashboards, bank ledgers, sorted reports.

---

## Full Flow Example

Table: users

| ID | Name  | Age |
|----|-------|-----|
| 1  | Alice | 23  |
| 2  | Bob   | 25  |
| 3  | Zara  | 21  |

We’ll now follow two flows:

Flow 1: Without Index (Full Table Scan)

Goal: SELECT * FROM users WHERE name = 'Zara';

1. Query hits DB engine
2. DB planner checks: ❌ No index available on name
3. DB opens the table file on disk (divided into pages)
4. Reads Page 1 → checks each row:
- Row 1 → name = Alice → skip
- Row 2 → name = Bob → skip
- Row 3 → name = Zara → ✅ match!

5. Return row to the user

Time: Slow for large tables

Disk: Reads every row = I/O heavy

Flow 2: With Index on name (B-Tree)

```sql
CREATE INDEX idx_name ON users(name);
```

Goal: SELECT * FROM users WHERE name = 'Zara';

1. Query hits DB engine
2. DB planner finds: ✅ Index available on name
3. Navigates B-Tree Index:
- Root → goes to leaf where Zara might be
4. Finds entry:
```pgsql
Zara → points to Row 3 (in page X)
```
5. DB jumps directly to Page X, Row 3
6. Returns row to the user

Time: Fast

Disk: Reads only 2 things:

- Part of the index
- Actual row in data file

# Table Storage Flow (Behind the Scenes)

When you insert:

```sql
INSERT INTO users VALUES (4, 'John', 26);
```
DB does:

1. Finds a data page with free space (or creates new page)
2. Adds Row 4 inside that page
3. Updates index (if any):
- Adds 'John' → Page Y, Row 2 to B-tree

## Data File (Disk)

```yaml
Page 1:
  - Row 1: Alice
  - Row 2: Bob
  - Row 3: Zara

Page 2:
  - Row 4: John
```
## Index (B-Tree)

```less
        [M]
       /   \
   [A-F]   [N-Z]
    |        |
  Alice     Zara → Page 1, Row 3  
  Bob       John → Page 2, Row 1
```

## Summary Flow Chart

```text
       Query: SELECT * FROM users WHERE name = 'Zara';
                            |
        +-------------------+-------------------+
        |                                       |
No Index?                                 Index Available?
  |                                             |
Full table scan                        Use B-Tree or Hash Index
  |                                             |
Check every row                          Go to Index Node
  |                                             |
Find Zara in row                          Jump to Disk Location
  |                                             |
Return result                              Return result
```
---

### Summary 

1. Table 

- A table is a logical collection of rows and columns.
- Physically, it's stored on disk as blocks/pages.
- Each row = one record (e.g., one user, one order).

2. Row_ID

- A unique identifier used internally by the database.
- Helps the database locate a specific row on disk.
- In some systems (like Oracle), RowID = block address + row offset.

3. Page

- Data on disk is stored in fixed-size chunks called pages (usually 4KB or 8KB).
- Each page contains multiple rows.
- Read/write always happens page-by-page, not row-by-row.

Example: If a page = 8KB, and each row = 1KB → one page stores 8 rows.

4. IO (Input/Output)

- Refers to reading/writing from disk.
- Disk IO is slow, so databases try to:
  - Use indexes to avoid full scans
  - Cache hot pages in memory (buffer pool)

5. Heap Data Structure
- Rows are stored unordered in the table.
- New rows are placed wherever there is free space.
- This is the default in many databases.

Advantage: Fast inserts

Disadvantage: Slower lookups (unless indexed)

6. Index Data Structure: B-Tree

- A balanced tree structure used to speed up lookups.
- Each node contains a range of values and pointers to child nodes or data.
- Helps DB jump directly to relevant data.

Example:
- Query: SELECT * FROM users WHERE name = 'Zara';
- Without index: full table scan
- With B-Tree: search tree → get Row ID → jump to page

7. Example of a Query Flow

```sql
SELECT * FROM users WHERE id = 101;
```

Without Index:
- DB checks every page → every row → match found
- Slower, especially on large tables

With B-Tree Index on id:
- DB looks in B-Tree → finds pointer to page
- Reads page directly → fetches row
- Much faster (fewer disk I/Os)

---

# Primary Key & Secondary (or Alternate/Secondary Index) Key

## ✅ Basic Difference

| Feature         | Primary Key                    | Secondary Key (Secondary Index)          |
|----------------|---------------------------------|-------------------------------------------|
| Uniqueness     | Must be unique                  | Can be unique or non-unique               |
| Null Allowed   | ❌ Not allowed                  | ✅ Allowed (if not unique)                |
| Default Index  | ✅ Automatically indexed        | ❌ Needs to be created manually           |
| One per table  | ✅ Only one                     | ✅ Can have multiple                      |

1. Primary Key Affects Physical Row Location (Clustered Index)

In many DB engines (e.g., MySQL InnoDB), the primary key is clustered — meaning:

- The actual data is stored sorted by the primary key.
- Other indexes point to the primary key value, not to the row directly.

So choosing a poor primary key (like UUID) can harm performance.

2. Secondary Indexes Use the Primary Key as a Pointer

```sql
CREATE TABLE users (
  id INT PRIMARY KEY,
  email VARCHAR(255),
  age INT
);

CREATE INDEX idx_email ON users(email);
```
Now, when you query:

```sql
SELECT * FROM users WHERE email = 'abc@example.com';
```
- The secondary index on email finds the matching row.
- But it doesn't store full row → it stores a pointer to the primary key (id)
- Then a second lookup is done to get the row via id.

This is called a "double lookup" – and it’s slower than accessing via primary key.

3. Updating Secondary Indexed Columns Is Costly

If a column has a secondary index and you UPDATE it:
- The DB must remove the old index entry
- Then insert the new index entry

Performance hit especially on frequent updates.

4. Secondary Indexes Can Be Covering Indexes

You can design secondary indexes to include extra columns:

```sql
CREATE INDEX idx_email_age ON users(email) INCLUDE (age);
```
Now, queries like:
```sql
SELECT age FROM users WHERE email = 'x';
```
Can be answered directly from the index (no row lookup needed).

5. NULL in Secondary Indexes
- Secondary indexes can index NULL values (unless set otherwise).
- Primary keys can never be NULL.

## Summary

| Feature                   | Primary Key                         | Secondary Key                            |
|---------------------------|--------------------------------------|-------------------------------------------|
| Must be Unique            | ✅ Yes                               | ❌ Not always                              |
| Allows NULL               | ❌ Never                             | ✅ Yes                                     |
| Auto-Indexed              | ✅ Yes                               | ❌ No (manual)                             |
| Affects Physical Row Order| ✅ Often (clustered)                 | ❌ No                                      |
| Lookup Speed              | ✅ Fast (no indirection)             | ❌ Needs extra step via primary key        |
| Best for                  | Identity, foreign keys               | Frequent queries on non-PK columns         |

--- 

# Database Pages

A page is the smallest unit of data storage and I/O in a database system.

- Typically fixed size: 4 KB, 8 KB, or 16 KB (depends on DB engine like PostgreSQL, MySQL, Oracle)
- A table’s rows are stored inside pages.
- When data is read/written, entire pages are loaded into memory, not individual rows.

Real-Life Analogy

Think of a notebook:
- Each page can contain multiple notes (rows)
- You can’t tear off just one word — you flip the whole page

A page usually contains:

```sql
+---------------------+
| Page Header         |  ← Metadata (page ID, free space, checksums)
+---------------------+
| Row 1               |
| Row 2               |
| ...                 |
+---------------------+
| Free Space          |  ← For inserts or updates
+---------------------+
| Row Directory       |  ← Offsets pointing to row positions
+---------------------+
```

### How Rows Are Stored in Pages

```sql 
CREATE TABLE users (
  id INT,
  name VARCHAR(50),
  age INT
);
```
You insert 3 rows:

```sql
INSERT INTO users VALUES (1, 'Alice', 23);
INSERT INTO users VALUES (2, 'Bob', 25);
INSERT INTO users VALUES (3, 'Zara', 21);
```
These rows are stored inside a page, like:

```pgsql
Page 1:
 - Row 1: id=1, name=Alice, age=23
 - Row 2: id=2, name=Bob, age=25
 - Row 3: id=3, name=Zara, age=21
```
If a page has space for 100 rows and you insert more than that, a new page is allocated.

### Why Pages Matter

1. Efficient I/O
- Instead of reading row-by-row, DB loads one page into memory — way faster.

2. Index Navigation
- Indexes point to pages + row offset, not the raw file location.

3. Update Strategy
- If a row grows too large (e.g., you update a TEXT column), the DB might:
  - Move it to a new page
  - Leave a pointer in the original page (called row forwarding)

## Page Types and What They Store

| Page Type       | Used For                                      | Example Data Stored                                      |
|------------------|-----------------------------------------------|-----------------------------------------------------------|
| Data Page        | Storing table rows (tuples)                  | `id=1, name='Alice', age=23`                             |
| Index Page       | Storing B-Tree or Hash index entries         | `name='Alice' → pointer to data row`                     |
| LOB Page         | Storing Large Objects (TEXT, BLOB, etc.)     | Large documents, images, long descriptions               |
| Undo Page        | Stores *before-image* of data for rollback   | Old value: `balance=500` before update                   |
| Redo/Log Page    | Write-Ahead Log (WAL) for crash recovery     | `"Row 3 was updated"` — can be replayed                  |
| Header Page      | Metadata about the table or segment          | Page count, row format, next pointers                    |
| Free Page        | Empty or reusable space                      | Available for new inserts                                |
| System Page      | Stores system-level info (transactions, etc.)| Transaction ID, page map, metadata                        |

## Detailed View by Data Type

| Data Type               | Stored In             | Notes                                               |
|--------------------------|------------------------|------------------------------------------------------|
| `INT`, `VARCHAR(50)`     | Data Page              | Stored inline unless large                          |
| `TEXT`, `BLOB`           | LOB Page               | Stored separately; main page stores pointer         |
| `JSON`, `XML`            | LOB or Data Page       | Depends on size and DB engine                       |
| Primary Key values       | Index Page (Clustered) | In clustered index, full row is here                |
| Secondary Index          | Index Page             | Contains key + pointer to primary key               |
| Old values (Rollback)    | Undo Page              | Used during transaction rollback                    |
| Crash recovery logs      | Redo Page (WAL)        | Replayed if crash happens                           |

# Flow Example: UPDATE a Row

```sql
UPDATE users SET age = 30 WHERE id = 1;
```
Page Usage:

1. Data Page:

- Original row is located and updated (age = 30)

2. Undo Page:

- Old value age = 23 is saved for rollback

3. Redo Page (WAL):

- Log entry recorded: "Row id=1 changed to age=30"

4. Index Page (if any):

- If age is indexed, update the index entry too

5. LOB Page (if row has large TEXT/BLOB):

- If update affects LOB, it updates the LOB page as well

---

All database engines use the same page types, sizes, or storage strategies — but the concept of pages is common to all major database engines. Here's how they differ and overlap:

### What’s Common Across All Engines?

- Page-Based Storage: All major RDBMS (PostgreSQL, MySQL, Oracle, SQL Server) use pages as the unit of I/O.

- Fixed Page Size: Pages are fixed size (e.g., 8KB), and data is read/written page-by-page, not row-by-row.

- Page Types: All engines have:

  - Data pages

  - Index pages

  - LOB pages (for large data)

  - Undo/Redo/Log pages

### What Differs Between Engines?

| Engine           | Default Page Size | Special Features                            | Notes                                           |
|------------------|-------------------|---------------------------------------------|------------------------------------------------|
| **PostgreSQL**   | 8 KB              | Uses **TOAST** for large values             | Stores oversized rows in separate pages       |
| **MySQL (InnoDB)**| 16 KB            | Has **undo, redo logs**, **compressed pages**| Uses clustered indexes for PK                  |
| **Oracle**       | 2 KB – 32 KB      | Advanced **tablespaces, row chaining**      | Many configurable page/block sizes            |
| **SQL Server**   | 8 KB              | Uses **IAM pages, allocation maps**         | Tracks which pages belong to which object     |
| **SQLite**       | 1 KB – 64 KB      | Uses **B-tree pages** even for data         | Embedded engine; simple structure             |


