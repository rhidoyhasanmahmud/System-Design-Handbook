# What Is a Transaction?

A **transaction** is a set of one or more operations treated as a **single unit of work**. It either **completes entirely (commit)** or **does nothing at all (rollback)** — ensuring data integrity.

---

## Why Use Transactions?

To guarantee your data stays **correct and consistent**, even if something goes wrong mid-process.

---

## Real-Life Analogy: ATM Money Transfer

- **Step 1:** Deduct money from Account A  
- **Step 2:** Add money to Account B  

If Step 1 succeeds but Step 2 fails, money disappears! Both steps must succeed **together or not at all**.

---

## SQL Example

```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;

-- If an error occurs, undo changes with:

ROLLBACK;
```
A transaction = safety shield for your data operations. It keeps your database consistent, reliable, and error-free.

---
## Transaction Life Cycle

Transactions move through several states during their lifecycle:

1. Active: The transaction is still in progress and has not yet been completed or committed.
2. Partially Committed: The transaction has finished executing all of its operations, but the changes haven’t yet been made permanent.
3. Committed: The transaction has been successfully completed, and its changes have been made permanent in the database.
4. Failed: The transaction encountered an error and cannot proceed. It will be rolled back.
5. Aborted: The transaction has been rolled back, undoing any changes made during the transaction.

![Transaction-Lifecycle](/images/Transaction-Lifecycle.PNG)
---

# ACID Properties

1. Atomicity - All or Nothing

A transaction is all or nothing. Either every operation in the transaction succeeds, or none do. If any step fails, the entire transaction is rolled back.

Transferring money from Account A to Account B:

- Deduct money from A
- Add money to B

If adding money to B fails, the deduction from A is undone.

```sql 
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;  -- Deduct
UPDATE accounts SET balance = balance + 100 WHERE id = 2;  -- Add

COMMIT;
-- If error occurs:
ROLLBACK;
```
Types:

- Atomic operations inside a transaction
- Partial operations don’t leave database in inconsistent state

2. Consistency - Valid Data Only

Every transaction takes the database from one valid state to another, respecting all rules (constraints, triggers, cascades).

If an account balance cannot be negative, a transaction that violates this rule will fail and rollback.

```sql
-- Trying to reduce balance below zero will fail due to constraint
UPDATE accounts SET balance = balance - 1000 WHERE id = 1;  -- If balance < 1000, rollback
```
Types:
- Data integrity rules (constraints, foreign keys, checks) ensure consistency

3. Isolation - No Dirty Reads

Each transaction is independent. It should not see partial results of another ongoing transaction.

Two users updating the same account balance simultaneously won’t corrupt the data. One transaction will wait for the other to finish or proceed safely.

```sql 
-- Transaction 1 reads balance 1000
-- Transaction 2 updates balance to 900
-- Transaction 1 reads again, must see either 1000 or 900, not partial updates
```

Isolation Levels: 

| Level              | Description                             | Side Effect           |
|--------------------|-----------------------------------------|------------------------|
| **Read Uncommitted** | Can read uncommitted data               | Dirty Read ✅          |
| **Read Committed**   | Reads only committed data               | Dirty Read ❌          |
| **Repeatable Read**  | Same query gives same result in one txn | Phantom Read ✅        |
| **Serializable**     | Fully isolated (like running one-by-one) | Best Isolation ✅     |

- Dirty Read: Reading data that hasn’t been committed yet. It may later be rolled back.
- Non-Repeatable Read: You run the same query twice in one transaction and get different results because someone else modified the data.
- Phantom Read: You run a query like ``` SELECT * FROM orders WHERE total > 1000```, and a new order gets inserted by another transaction. Now if you run the query again, you get a different result — new rows have “appeared like ghosts.”

4. Durability - It Stays Forever

Once a transaction is committed, it is permanently saved, even if the system crashes.

You transfer money at 1:00 PM
- Power cut at 1:01 PM
- When server restarts — the transaction is still there!

Databases use:

- Write-Ahead Logs (WAL) – to ensure durability
- Checkpointing

## Summary

| Property   | What it Ensures                     | Real-Life Analogy                             |
|------------|-------------------------------------|------------------------------------------------|
| Atomicity  | All steps done or none              | Sending parcel – either delivered or returned  |
| Consistency| Valid data only                     | No student has -5 age                          |
| Isolation  | No interference between transactions| One person at ATM at a time                    |
| Durability | Data is permanent after commit      | Receipt remains even after power failure       |

--- 

#  Immediate Consistency (Strong Consistency)

As soon as you write data, all subsequent reads reflect that change instantly — from any node or client.

You update your profile picture on Facebook, and everyone sees the new photo right away — no one sees the old one.

```text
Write: name = 'Hasan' to Node A
Immediately Read from Node B → returns 'Hasan'
```

Guarantees:
- Always the latest data

Downsides
- Slower writes (wait for sync across nodes)
- Harder to scale

###  Where It’s Used:
- Traditional RDBMS like PostgreSQL, MySQL (single-node)
- Distributed systems with strong consistency mode (e.g., Google Spanner)
---

# Eventual Consistency

After a write, data may not be immediately visible everywhere. But given time, all nodes will eventually sync and reflect the latest state.

You post a tweet. Some friends see it instantly, others see it after a few seconds. Eventually, everyone sees it.

```text
Write: name = 'Hasan' to Node A
Immediately Read from Node B → might return old value
Later Read → returns 'Hasan'
```

Pros:
- Fast write performance
- High availability

Cons:
- You may get stale (old) data temporarily

Where It’s Used:
- Amazon DynamoDB (default)
- Apache Cassandra
- MongoDB (default for some setups)

## Immediate vs Eventual Consistency

| Feature            | Immediate Consistency         | Eventual Consistency              |
|-------------------|-------------------------------|----------------------------------|
| **Read Accuracy** | Always latest                 | May be outdated                   |
| **Speed (Write)** | Slower                        | Faster                            |
| **Use Case**      | Banking, finance              | Social media, analytics           |
| **Example Systems** | SQL, Google Spanner          | DynamoDB, Cassandra, MongoDB      |
| **Availability**  | May reduce availability       | Highly available                  |
| **Complexity**    | Higher                        | Lower                             |

## Real-World Comparison: Immediate vs Eventual Consistency

| Application Type        | Recommended Consistency                    |
|-------------------------|--------------------------------------------|
| Bank transactions     | Immediate (you want accurate balance)       |
| Social media likes    | Eventual (okay if counts are briefly off)   |
| Shopping cart updates | Immediate (critical to avoid mismatch)      |
| Analytics dashboard   | Eventual (data can be a few seconds late)   |
