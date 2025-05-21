# What Is Database Indexing?

A database index is like a table of contents for your data. It helps the database find rows faster, just like how a book index helps you find topics quickly without reading every page.

# Why Use Indexes?

Without an index:

- A query must scan every row in the table → Full Table Scan
- Slow for large tables

With an index:

- DB engine can jump to the data it needs → Fast Lookup

# Real-Life Analogy

Imagine a library:

| Book Search Type         | Time Taken |
|--------------------------|------------|
| 🔍 Search every shelf     | 🕒 Slow     |
| 📘 Use book index (TOC)   | ⚡ Fast     |

Same with a database — indexes speed up searching.

## Types of Indexes

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

## How Indexes Work (Flow)

```text
SELECT * FROM users WHERE name = 'Zara';

1. Query sent to DB
2. DB checks if 'name' is indexed
3. If indexed → uses B-Tree to find value
4. Finds pointer to row
5. Fetches row from table page
```
Without index → DB would read every row.

## When NOT to Use Indexes

| Situation                                | Why Not Index?                          |
|------------------------------------------|------------------------------------------|
| 🟦 Columns frequently updated             | Index needs constant maintenance         |
| 📏 Small tables                          | Full scan may be faster                  |
| 📊 Low-cardinality columns (e.g., gender)| Index not selective enough               |
| 🗒️ Heavy insert load                      | Index slows down insert speed            |

## Best Practices
1. Index columns used in WHERE, JOIN, ORDER BY
2. Use composite indexes wisely (order matters!)
3. Don’t over-index — hurts write performance
4. Monitor with EXPLAIN / Query Plan

