# Data Manipulation Language (DML) - Complete Guide

## Index
1. [Introduction](#introduction)
2. [INSERT Statement](#insert-statement)
   - [Single Row Insert](#single-row-insert)
   - [Multiple Row Insert](#multiple-row-insert)
   - [INSERT with SELECT](#insert-with-select)
   - [INSERT with Default Values](#insert-with-default-values)
3. [UPDATE Statement](#update-statement)
   - [Simple UPDATE](#simple-update)
   - [UPDATE with WHERE](#update-with-where)
   - [UPDATE Multiple Columns](#update-multiple-columns)
   - [UPDATE with JOIN](#update-with-join)
4. [DELETE Statement](#delete-statement)
   - [DELETE with WHERE](#delete-with-where)
   - [DELETE All Rows](#delete-all-rows)
   - [DELETE with JOIN](#delete-with-join)
5. [MERGE / UPSERT](#merge--upsert)
   - [MERGE Statement](#merge-statement)
   - [INSERT ON CONFLICT (PostgreSQL)](#insert-on-conflict-postgresql)
   - [REPLACE INTO (MySQL)](#replace-into-mysql)
6. [Transactions](#transactions)
   - [BEGIN Transaction](#begin-transaction)
   - [COMMIT](#commit)
   - [ROLLBACK](#rollback)
   - [SAVEPOINT](#savepoint)
   - [Transaction Properties (ACID)](#transaction-properties-acid)
7. [Best Practices](#best-practices)
8. [Common Interview Questions](#common-interview-questions)

---

## Introduction

**DML (Data Manipulation Language)** consists of SQL statements used to manipulate data in database tables.

**Main DML Operations:**
- **INSERT**: Add new rows
- **UPDATE**: Modify existing rows
- **DELETE**: Remove rows
- **MERGE/UPSERT**: Insert or update based on conditions

**Key Characteristics:**
- Changes can be rolled back (with transactions)
- Affects table data, not structure
- Can be used with WHERE clauses for selective operations
- Subject to constraints (PRIMARY KEY, FOREIGN KEY, CHECK, etc.)

---

## INSERT Statement

The **INSERT** statement adds new rows to a table.

### Single Row Insert

Insert one row by specifying values for each column.

**Syntax:**
```sql
INSERT INTO table_name (column1, column2, column3)
VALUES (value1, value2, value3);
```

**Example:**

```sql
INSERT INTO employees (id, name, department, salary)
VALUES (1, 'Alice', 'Sales', 60000);
```

**Before - employees table:**

| id | name | department | salary |
|----|------|------------|--------|
| *(empty)* |

**After INSERT:**

| id | name | department | salary |
|----|------|------------|--------|
| 1 | Alice | Sales | 60000 |

*Explanation:* A new row is added with the specified values.

---

### Multiple Row Insert

Insert multiple rows in a single statement.

**Syntax:**
```sql
INSERT INTO table_name (column1, column2)
VALUES 
    (value1, value2),
    (value3, value4),
    (value5, value6);
```

**Example:**

```sql
INSERT INTO employees (id, name, department, salary)
VALUES 
    (1, 'Alice', 'Sales', 60000),
    (2, 'Bob', 'IT', 75000),
    (3, 'Carol', 'HR', 55000);
```

**Before - employees table:**

| id | name | department | salary |
|----|------|------------|--------|
| *(empty)* |

**After INSERT:**

| id | name | department | salary |
|----|------|------------|--------|
| 1 | Alice | Sales | 60000 |
| 2 | Bob | IT | 75000 |
| 3 | Carol | HR | 55000 |

*Explanation:* Three rows inserted in a single operation (more efficient than three separate INSERTs).

---

### INSERT with SELECT

Insert data from another table or query result.

**Syntax:**
```sql
INSERT INTO table_name (column1, column2)
SELECT column1, column2
FROM source_table
WHERE condition;
```

**Example:**

```sql
INSERT INTO high_performers (emp_id, emp_name, emp_salary)
SELECT id, name, salary
FROM employees
WHERE salary > 65000;
```

**Source - employees table:**

| id | name | salary |
|----|------|--------|
| 1 | Alice | 60000 |
| 2 | Bob | 75000 |
| 3 | Carol | 70000 |

**Target - high_performers table (After INSERT):**

| emp_id | emp_name | emp_salary |
|--------|----------|------------|
| 2 | Bob | 75000 |
| 3 | Carol | 70000 |

*Explanation:* Rows from employees with salary > 65000 are copied to high_performers table.

---

### INSERT with Default Values

Insert rows using default values defined in table schema.

**Example:**

```sql
-- Table with default values
CREATE TABLE orders (
    id INT PRIMARY KEY,
    order_date DATE DEFAULT CURRENT_DATE,
    status VARCHAR(20) DEFAULT 'pending'
);

-- Insert using defaults
INSERT INTO orders (id) VALUES (1);

-- Insert overriding one default
INSERT INTO orders (id, status) VALUES (2, 'confirmed');
```

**After INSERT:**

| id | order_date | status |
|----|------------|---------|
| 1 | 2024-02-05 | pending |
| 2 | 2024-02-05 | confirmed |

*Explanation:* Column `order_date` uses DEFAULT (current date). Row 1 uses default status, Row 2 overrides it.

---

## UPDATE Statement

The **UPDATE** statement modifies existing rows in a table.

### Simple UPDATE

Update all rows in a table.

**Syntax:**
```sql
UPDATE table_name
SET column1 = value1;
```

**Example:**

```sql
UPDATE employees
SET department = 'General';
```

**Before:**

| id | name | department |
|----|------|------------|
| 1 | Alice | Sales |
| 2 | Bob | IT |

**After UPDATE:**

| id | name | department |
|----|------|------------|
| 1 | Alice | General |
| 2 | Bob | General |

*Explanation:* **All rows** updated. ⚠️ Use with caution!

---

### UPDATE with WHERE

Update specific rows based on condition.

**Syntax:**
```sql
UPDATE table_name
SET column1 = value1
WHERE condition;
```

**Example:**

```sql
UPDATE employees
SET salary = salary * 1.10
WHERE department = 'Sales';
```

**Before:**

| id | name | department | salary |
|----|------|------------|--------|
| 1 | Alice | Sales | 60000 |
| 2 | Bob | IT | 75000 |
| 3 | Carol | Sales | 55000 |

**After UPDATE:**

| id | name | department | salary |
|----|------|------------|--------|
| 1 | Alice | Sales | 66000 |
| 2 | Bob | IT | 75000 |
| 3 | Carol | Sales | 60500 |

*Explanation:* Only Sales department employees get 10% raise. Bob (IT) unchanged.

---

### UPDATE Multiple Columns

Update several columns in one statement.

**Example:**

```sql
UPDATE employees
SET salary = 80000,
    department = 'Senior IT'
WHERE id = 2;
```

**Before:**

| id | name | department | salary |
|----|------|------------|--------|
| 2 | Bob | IT | 75000 |

**After UPDATE:**

| id | name | department | salary |
|----|------|------------|--------|
| 2 | Bob | Senior IT | 80000 |

*Explanation:* Both salary and department updated for Bob.

---

### UPDATE with JOIN

Update based on data from another table.

**Example:**

```sql
UPDATE employees e
SET e.bonus = d.bonus_rate * e.salary
FROM departments d
WHERE e.department_id = d.id;
```

**Before - employees table:**

| id | name | salary | department_id | bonus |
|----|------|--------|---------------|-------|
| 1 | Alice | 60000 | 10 | NULL |
| 2 | Bob | 75000 | 20 | NULL |

**departments table:**

| id | dept_name | bonus_rate |
|----|-----------|------------|
| 10 | Sales | 0.10 |
| 20 | IT | 0.15 |

**After UPDATE:**

| id | name | salary | department_id | bonus |
|----|------|--------|---------------|-------|
| 1 | Alice | 60000 | 10 | 6000 |
| 2 | Bob | 75000 | 20 | 11250 |

*Explanation:* Bonus calculated using bonus_rate from departments table.

---

## DELETE Statement

The **DELETE** statement removes rows from a table.

### DELETE with WHERE

Delete specific rows based on condition.

**Syntax:**
```sql
DELETE FROM table_name
WHERE condition;
```

**Example:**

```sql
DELETE FROM employees
WHERE department = 'Temp';
```

**Before:**

| id | name | department |
|----|------|------------|
| 1 | Alice | Sales |
| 2 | Bob | Temp |
| 3 | Carol | IT |
| 4 | David | Temp |

**After DELETE:**

| id | name | department |
|----|------|------------|
| 1 | Alice | Sales |
| 3 | Carol | IT |

*Explanation:* Only rows where department = 'Temp' are deleted.

---

### DELETE All Rows

Remove all rows from a table (structure remains).

**Syntax:**
```sql
DELETE FROM table_name;
```

**Example:**

```sql
DELETE FROM temp_data;
```

**Before:**

| id | data |
|----|------|
| 1 | A |
| 2 | B |

**After DELETE:**

| id | data |
|----|------|
| *(empty)* |

*Explanation:* All rows deleted. Table structure remains. **Use TRUNCATE for better performance.**

---

### DELETE with JOIN

Delete based on related table data.

**Example:**

```sql
DELETE e
FROM employees e
JOIN departments d ON e.department_id = d.id
WHERE d.status = 'closed';
```

**Before - employees table:**

| id | name | department_id |
|----|------|---------------|
| 1 | Alice | 10 |
| 2 | Bob | 20 |
| 3 | Carol | 10 |

**departments table:**

| id | dept_name | status |
|----|-----------|--------|
| 10 | Old Dept | closed |
| 20 | Active | open |

**After DELETE:**

| id | name | department_id |
|----|------|---------------|
| 2 | Bob | 20 |

*Explanation:* Employees in 'closed' departments are deleted.

---

## MERGE / UPSERT

**MERGE/UPSERT** operations insert new rows or update existing ones based on a condition.

### MERGE Statement

Standard SQL MERGE (supported in SQL Server, Oracle, PostgreSQL 15+).

**Theory:**
- Combines INSERT and UPDATE in one statement
- Based on matching condition
- Three possible actions: INSERT, UPDATE, DELETE

**Syntax:**
```sql
MERGE INTO target_table AS target
USING source_table AS source
ON target.id = source.id
WHEN MATCHED THEN
    UPDATE SET target.column = source.column
WHEN NOT MATCHED THEN
    INSERT (column1, column2) VALUES (source.column1, source.column2);
```

**Example:**

```sql
MERGE INTO inventory AS target
USING new_stock AS source
ON target.product_id = source.product_id
WHEN MATCHED THEN
    UPDATE SET target.quantity = target.quantity + source.quantity
WHEN NOT MATCHED THEN
    INSERT (product_id, quantity) VALUES (source.product_id, source.quantity);
```

**Before - inventory table:**

| product_id | quantity |
|------------|----------|
| 101 | 50 |
| 102 | 30 |

**new_stock table:**

| product_id | quantity |
|------------|----------|
| 101 | 20 |
| 103 | 40 |

**After MERGE:**

| product_id | quantity |
|------------|----------|
| 101 | 70 |
| 102 | 30 |
| 103 | 40 |

*Explanation:* Product 101 quantity updated (50+20=70). Product 103 inserted as new. Product 102 unchanged.

---

### INSERT ON CONFLICT (PostgreSQL)

PostgreSQL's UPSERT syntax.

**Syntax:**
```sql
INSERT INTO table_name (column1, column2)
VALUES (value1, value2)
ON CONFLICT (conflict_column)
DO UPDATE SET column2 = EXCLUDED.column2;
```

**Example:**

```sql
INSERT INTO users (id, username, login_count)
VALUES (1, 'alice', 1)
ON CONFLICT (id)
DO UPDATE SET login_count = users.login_count + 1;
```

**Before - users table:**

| id | username | login_count |
|----|----------|-------------|
| 1 | alice | 5 |

**After executing twice:**

| id | username | login_count |
|----|----------|-------------|
| 1 | alice | 6 |

*Explanation:* First execution: conflict on id=1, updates login_count to 6. Username unchanged.

---

### REPLACE INTO (MySQL)

MySQL's UPSERT syntax (deletes then inserts).

**Syntax:**
```sql
REPLACE INTO table_name (column1, column2)
VALUES (value1, value2);
```

**Example:**

```sql
REPLACE INTO settings (key_name, value)
VALUES ('theme', 'dark');
```

**Before - settings table:**

| key_name | value |
|----------|-------|
| theme | light |

**After REPLACE:**

| key_name | value |
|----------|-------|
| theme | dark |

*Explanation:* If key_name='theme' exists, row is deleted and new row inserted. ⚠️ Triggers DELETE and INSERT events.

---

## Transactions

**Transactions** group multiple DML statements into a single unit of work that either completely succeeds or completely fails.

### BEGIN Transaction

Start a transaction block.

**Syntax:**
```sql
BEGIN;  -- or START TRANSACTION;
```

**Example:**

```sql
BEGIN;

INSERT INTO accounts (id, balance) VALUES (1, 1000);
INSERT INTO accounts (id, balance) VALUES (2, 500);
```

*Explanation:* Transaction started. Changes not yet permanent.

---

### COMMIT

**COMMIT** makes all changes in the transaction permanent.

**Syntax:**
```sql
COMMIT;
```

**Example:**

```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;
```

**Before Transaction:**

| id | balance |
|----|---------|
| 1 | 1000 |
| 2 | 500 |

**After COMMIT:**

| id | balance |
|----|---------|
| 1 | 900 |
| 2 | 600 |

*Explanation:* Both updates succeed together. Money transferred from account 1 to account 2.

---

### ROLLBACK

**ROLLBACK** undoes all changes in the transaction.

**Syntax:**
```sql
ROLLBACK;
```

**Example:**

```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Oops, error detected!
ROLLBACK;
```

**Before Transaction:**

| id | balance |
|----|---------|
| 1 | 1000 |
| 2 | 500 |

**After ROLLBACK:**

| id | balance |
|----|---------|
| 1 | 1000 |
| 2 | 500 |

*Explanation:* All changes discarded. Database returns to state before BEGIN.

---

### SAVEPOINT

Create checkpoints within a transaction for partial rollback.

**Syntax:**
```sql
SAVEPOINT savepoint_name;
ROLLBACK TO savepoint_name;
```

**Example:**

```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;  -- Change 1
SAVEPOINT after_debit;

UPDATE accounts SET balance = balance + 100 WHERE id = 2;  -- Change 2
SAVEPOINT after_credit;

UPDATE accounts SET balance = balance - 50 WHERE id = 3;   -- Change 3 (error!)

-- Rollback only Change 3
ROLLBACK TO after_credit;

COMMIT;  -- Commits Change 1 and 2 only
```

**Before Transaction:**

| id | balance |
|----|---------|
| 1 | 1000 |
| 2 | 500 |
| 3 | 300 |

**After COMMIT:**

| id | balance |
|----|---------|
| 1 | 900 |
| 2 | 600 |
| 3 | 300 |

*Explanation:* Change 3 rolled back to savepoint. Changes 1 and 2 committed.

---

### Transaction Properties (ACID)

Transactions must satisfy **ACID** properties:

| Property | Description | Example |
|----------|-------------|---------|
| **Atomicity** | All or nothing - transaction fully succeeds or fully fails | Transfer money: both debit and credit succeed, or neither |
| **Consistency** | Database moves from valid state to valid state | Balance never negative; constraints satisfied |
| **Isolation** | Concurrent transactions don't interfere | Two transfers don't see each other's intermediate states |
| **Durability** | Committed changes persist even after system failure | After COMMIT, data survives crash |

**Example - Bank Transfer:**

```sql
BEGIN;

-- Debit from Account 1
UPDATE accounts 
SET balance = balance - 500 
WHERE id = 1 AND balance >= 500;  -- Check sufficient funds

-- Credit to Account 2
UPDATE accounts 
SET balance = balance + 500 
WHERE id = 2;

-- Verify both updates succeeded
IF @@ROWCOUNT = 2 THEN
    COMMIT;
ELSE
    ROLLBACK;  -- Insufficient funds or error
END IF;
```

**Scenario 1 - Success:**

**Before:**
| id | balance |
|----|---------|
| 1 | 1000 |
| 2 | 500 |

**After COMMIT:**
| id | balance |
|----|---------|
| 1 | 500 |
| 2 | 1000 |

**Scenario 2 - Failure (Insufficient Funds):**

**Before:**
| id | balance |
|----|---------|
| 1 | 300 |
| 2 | 500 |

**After ROLLBACK:**
| id | balance |
|----|---------|
| 1 | 300 |
| 2 | 500 |

*Explanation:* ACID ensures money isn't lost or created. Either both succeed or both fail.

---

## Best Practices

### INSERT Best Practices
1. **Always specify column names** - Don't rely on column order
2. **Use multi-row INSERT** for bulk operations - More efficient
3. **Validate data** before inserting - Check constraints
4. **Use transactions** for related inserts - Ensure consistency

### UPDATE Best Practices
5. **Always use WHERE clause** - Unless you really want to update all rows
6. **Test with SELECT first** - Verify which rows will be affected
   ```sql
   -- Test first
   SELECT * FROM employees WHERE department = 'Sales';
   -- Then update
   UPDATE employees SET salary = salary * 1.10 WHERE department = 'Sales';
   ```
7. **Use transactions** for critical updates - Allow rollback if needed
8. **Backup before mass updates** - Safety net for mistakes

### DELETE Best Practices
9. **Always use WHERE clause** - Avoid accidental data loss
10. **Use transactions** - Enable rollback
11. **Consider soft deletes** - Mark as deleted instead of removing
    ```sql
    UPDATE employees SET is_deleted = TRUE WHERE id = 5;
    ```
12. **Check foreign keys** - Understand cascade behavior

### Transaction Best Practices
13. **Keep transactions short** - Minimize lock time
14. **Use appropriate isolation level** - Balance consistency and performance
15. **Always handle errors** - Implement proper ROLLBACK logic
16. **Avoid user interaction** in transactions - Don't wait for input
17. **Use SAVEPOINT** for complex transactions - Partial rollback capability

### General DML Best Practices
18. **Use explicit transactions** for related operations
19. **Log important DML operations** - Audit trail
20. **Test in development** first - Never test in production
21. **Understand indexes** - DML can be slow on heavily indexed tables
22. **Monitor performance** - Watch for blocking and deadlocks

---

## Common Interview Questions

### Q1: What's the difference between DELETE and TRUNCATE?

**Answer:**

| Feature | DELETE | TRUNCATE |
|---------|--------|----------|
| **Type** | DML | DDL |
| **WHERE clause** | Supported | Not supported |
| **Rollback** | Can rollback | Cannot rollback (in most DBs) |
| **Speed** | Slower | Faster |
| **Triggers** | Fires triggers | Doesn't fire triggers |
| **Identity reset** | Doesn't reset | Resets auto-increment |

**Example:**
```sql
DELETE FROM logs WHERE date < '2024-01-01';  -- Deletes specific rows
TRUNCATE TABLE logs;                          -- Removes all rows, resets ID
```

---

### Q2: How do you prevent duplicate inserts?

**Answer:**

**Method 1: UNIQUE constraint**
```sql
CREATE TABLE users (
    email VARCHAR(100) UNIQUE
);
-- INSERT fails if email exists
```

**Method 2: INSERT ON CONFLICT (PostgreSQL)**
```sql
INSERT INTO users (email, name)
VALUES ('alice@example.com', 'Alice')
ON CONFLICT (email) DO NOTHING;
```

**Method 3: Check before INSERT**
```sql
INSERT INTO users (email, name)
SELECT 'alice@example.com', 'Alice'
WHERE NOT EXISTS (
    SELECT 1 FROM users WHERE email = 'alice@example.com'
);
```

---

### Q3: What happens if UPDATE fails midway?

**Answer:**

**Without Transaction:**
- Partial updates committed
- Database in inconsistent state
- Cannot rollback

**With Transaction:**
- All changes rolled back automatically on error
- Database remains consistent
- Can handle error gracefully

```sql
BEGIN;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
    -- If second UPDATE fails, first UPDATE is also rolled back
COMMIT;
```

---

### Q4: How to update one table based on another?

**Answer:**

```sql
-- Method 1: UPDATE with JOIN
UPDATE employees e
SET e.salary = s.new_salary
FROM salary_updates s
WHERE e.id = s.emp_id;

-- Method 2: UPDATE with subquery
UPDATE employees
SET salary = (
    SELECT new_salary 
    FROM salary_updates 
    WHERE emp_id = employees.id
)
WHERE id IN (SELECT emp_id FROM salary_updates);
```

---

### Q5: Explain ACID with an example

**Answer:**

**Example: Online Shopping Order**

```sql
BEGIN;

-- 1. Reduce inventory (Atomicity)
UPDATE products SET stock = stock - 1 WHERE id = 101;

-- 2. Create order (Atomicity)
INSERT INTO orders (user_id, product_id, total) 
VALUES (1, 101, 99.99);

-- 3. Charge payment (Atomicity)
UPDATE user_balance SET balance = balance - 99.99 WHERE user_id = 1;

-- All 3 steps succeed together or all fail together
COMMIT;  -- Durability: Changes persist even if server crashes
```

- **Atomicity**: All 3 steps succeed or none
- **Consistency**: Stock never negative, balance accurate
- **Isolation**: Other users don't see partial state
- **Durability**: After COMMIT, order survives system crash

---

## Summary

### DML Operations Summary

| Operation | Purpose | Reversible | Affects |
|-----------|---------|------------|---------|
| **INSERT** | Add new rows | Yes (with transaction) | Adds data |
| **UPDATE** | Modify existing rows | Yes (with transaction) | Changes data |
| **DELETE** | Remove rows | Yes (with transaction) | Removes data |
| **MERGE/UPSERT** | Insert or update | Yes (with transaction) | Adds or changes data |

### Transaction Control Summary

| Command | Purpose | When to Use |
|---------|---------|-------------|
| **BEGIN** | Start transaction | Before related DML operations |
| **COMMIT** | Save changes | When all operations succeed |
| **ROLLBACK** | Undo changes | When error occurs |
| **SAVEPOINT** | Create checkpoint | Complex transactions with partial rollback needs |

### Key Takeaways

1. **Always use WHERE** with UPDATE/DELETE (unless intentional)
2. **Use transactions** for related operations
3. **Test with SELECT** before UPDATE/DELETE
4. **MERGE/UPSERT** simplifies conditional insert/update
5. **ACID properties** ensure data integrity
6. **Backup before** mass data changes
7. **Understand rollback** limitations (DDL vs DML)

**Master DML and transactions to handle data safely and efficiently!** 🎯
