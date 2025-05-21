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

1. Atomicity

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

2. Consistency

Every transaction takes the database from one valid state to another, respecting all rules (constraints, triggers, cascades).

If an account balance cannot be negative, a transaction that violates this rule will fail and rollback.

```sql
-- Trying to reduce balance below zero will fail due to constraint
UPDATE accounts SET balance = balance - 1000 WHERE id = 1;  -- If balance < 1000, rollback
```
Types:
- Data integrity rules (constraints, foreign keys, checks) ensure consistency

3. Isolation

Concurrent transactions do not interfere with each other. Intermediate states of a transaction are invisible to others until committed.

Two users updating the same account balance simultaneously won’t corrupt the data. One transaction will wait for the other to finish or proceed safely.

```sql 
-- Transaction 1 reads balance 1000
-- Transaction 2 updates balance to 900
-- Transaction 1 reads again, must see either 1000 or 900, not partial updates
```
Types (Isolation Levels):

> Phantom Read: When you check something in the database twice during a transaction, and new rows appear or disappear in the second check because someone else added or deleted them.



