# SQL Complete Revision Guide
## Your Ultimate SQL Reference & Interview Preparation

---

## 📚 Table of Contents

### Part 0: Core Theory & Concepts
0. [SQL Theory Foundation](#0-sql-theory-foundation)

### Part 1: Fundamentals
1. [SQL Basics](#1-sql-basics)
2. [SELECT & Filtering](#2-select--filtering)
3. [Operators](#3-operators)

### Part 2: Data Retrieval
4. [SQL Joins](#4-sql-joins)
5. [Aggregations & Grouping](#5-aggregations--grouping)

### Part 3: Advanced Queries
6. [Subqueries & CTEs](#6-subqueries--ctes)
7. [Window Functions](#7-window-functions)

### Part 4: Data Manipulation
8. [DML Operations](#8-dml-operations)
9. [Transactions](#9-transactions)

### Part 5: Database Design
10. [Keys & Constraints](#10-keys--constraints)
11. [Normalization](#11-normalization)
12. [Relationships](#12-relationships)

### Part 6: Performance
13. [Indexes & Optimization](#13-indexes--optimization)
14. [Query Optimization](#14-query-optimization)

### Part 7: Interview Patterns
15. [Common Interview Questions](#15-common-interview-questions)
16. [Real-World Patterns](#16-real-world-patterns)

---

# Part 0: Core Theory & Concepts

## 0. SQL Theory Foundation

### What is a Database?

**Definition:** A database is an organized collection of structured data stored electronically in a computer system.

**Purpose:**
- Store data persistently
- Organize data efficiently
- Enable quick retrieval
- Support multiple users
- Ensure data integrity

**Types of Databases:**

| Type | Description | Example |
|------|-------------|---------|
| **Relational** | Data in tables with relationships | MySQL, PostgreSQL, Oracle |
| **NoSQL** | Non-relational, flexible schema | MongoDB, Cassandra |
| **Graph** | Nodes and edges | Neo4j |
| **Time-Series** | Time-stamped data | InfluxDB |

### The Relational Model

**Core Principles:**

**1. Data Organization:**
- Data organized in **tables** (relations)
- Each table represents an **entity** (thing/concept)
- Tables have **rows** (records) and **columns** (attributes)

**2. Mathematical Foundation:**
- Based on set theory and predicate logic
- Tables are sets of tuples (rows)
- Operations follow relational algebra

**3. Relationships:**
- Tables connect through **keys**
- **Primary Key:** Unique identifier for each row
- **Foreign Key:** Reference to another table's primary key

**Visual Model:**
```
Database: Company
├── Table: Employees
│   ├── Columns: id, name, dept_id, salary
│   └── Rows: Individual employee records
├── Table: Departments
│   ├── Columns: dept_id, dept_name, location
│   └── Rows: Individual department records
└── Relationship: dept_id connects them
```

### Why SQL?

**Advantages:**
1. **Declarative:** Say WHAT you want, not HOW to get it
2. **Standardized:** Works across different databases (mostly)
3. **Powerful:** Complex operations in simple syntax
4. **Optimized:** Database engine handles optimization
5. **Mature:** Decades of development and refinement

**SQL vs Programming Languages:**

| Aspect | SQL | Python/Java |
|--------|-----|-------------|
| **Paradigm** | Declarative | Imperative |
| **Focus** | What to retrieve | How to retrieve |
| **Optimization** | Automatic | Manual |
| **Data handling** | Set-based | Record-by-record |

**Example Comparison:**
```sql
-- SQL (declarative)
SELECT name FROM employees WHERE salary > 50000;

-- Pseudo-code (imperative)
results = []
for employee in employees:
    if employee.salary > 50000:
        results.append(employee.name)
return results
```

### RDBMS Architecture

**Layered Architecture:**

```
┌─────────────────────────────────────┐
│     Application Layer               │
│  (Your SQL queries)                 │
├─────────────────────────────────────┤
│     SQL Parser & Optimizer          │
│  (Converts SQL to execution plan)   │
├─────────────────────────────────────┤
│     Query Execution Engine          │
│  (Executes the plan)                │
├─────────────────────────────────────┤
│     Transaction Manager             │
│  (ACID compliance)                  │
├─────────────────────────────────────┤
│     Storage Engine                  │
│  (Manages physical storage)         │
├─────────────────────────────────────┤
│     Physical Storage                │
│  (Disk/SSD)                         │
└─────────────────────────────────────┘
```

**How a Query Executes:**

1. **Parsing:** SQL syntax validated
2. **Optimization:** Query optimizer finds best execution plan
3. **Execution:** Plan executed step by step
4. **Result:** Data returned to application

### ACID Properties - Deep Dive

**A - Atomicity**

**Theory:** Transactions are indivisible units of work.

**Principle:** All operations in a transaction succeed together, or all fail together. No partial completion.

**Real-world Analogy:** Bank transfer
- Debit from Account A
- Credit to Account B
- Both must happen, or neither

**Example:**
```sql
BEGIN TRANSACTION;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;  -- Debit
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;  -- Credit
COMMIT;  -- Both succeed
-- If either fails, ROLLBACK happens automatically
```

**Without Atomicity:** Money could disappear (debited but not credited) or be created (credited but not debited).

---

**C - Consistency**

**Theory:** Database moves from one valid state to another valid state.

**Principle:** All constraints, triggers, and rules are satisfied before and after transaction.

**Real-world Analogy:** Inventory system
- Stock count never negative
- Total items in = Total items out

**Example:**
```sql
-- Constraint ensures consistency
ALTER TABLE products ADD CONSTRAINT chk_stock CHECK (stock >= 0);

-- This transaction would fail
UPDATE products SET stock = stock - 100 WHERE product_id = 1;
-- Fails if stock becomes negative
```

**Without Consistency:** Database could have invalid states (negative stock, broken foreign keys, violated constraints).

---

**I - Isolation**

**Theory:** Concurrent transactions don't interfere with each other.

**Principle:** Each transaction executes as if it's the only transaction running, even when many run simultaneously.

**Isolation Levels:**

| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|-------|------------|---------------------|--------------|
| **Read Uncommitted** | ✓ Possible | ✓ Possible | ✓ Possible |
| **Read Committed** | ✗ Prevented | ✓ Possible | ✓ Possible |
| **Repeatable Read** | ✗ Prevented | ✗ Prevented | ✓ Possible |
| **Serializable** | ✗ Prevented | ✗ Prevented | ✗ Prevented |

**Problems Without Isolation:**

1. **Dirty Read:** Reading uncommitted changes
```sql
-- Transaction 1
UPDATE accounts SET balance = 1000 WHERE id = 1;
-- Not committed yet

-- Transaction 2 (reads dirty data)
SELECT balance FROM accounts WHERE id = 1;  -- Sees 1000

-- Transaction 1
ROLLBACK;  -- Transaction 2 read data that never existed!
```

2. **Non-Repeatable Read:** Same query returns different results
```sql
-- Transaction 1
SELECT balance FROM accounts WHERE id = 1;  -- Returns 1000

-- Transaction 2
UPDATE accounts SET balance = 2000 WHERE id = 1;
COMMIT;

-- Transaction 1 (same query, different result)
SELECT balance FROM accounts WHERE id = 1;  -- Returns 2000!
```

3. **Phantom Read:** New rows appear between queries
```sql
-- Transaction 1
SELECT COUNT(*) FROM orders WHERE status = 'pending';  -- Returns 5

-- Transaction 2
INSERT INTO orders (status) VALUES ('pending');
COMMIT;

-- Transaction 1 (same query, different count)
SELECT COUNT(*) FROM orders WHERE status = 'pending';  -- Returns 6!
```

---

**D - Durability**

**Theory:** Committed transactions persist permanently.

**Principle:** Once COMMIT succeeds, changes survive system crashes, power failures, or errors.

**How it Works:**
1. **Write-Ahead Logging (WAL):** Changes written to log first
2. **Checkpoint:** Periodically flush to disk
3. **Recovery:** Replay log after crash

**Example:**
```sql
BEGIN TRANSACTION;
    INSERT INTO orders VALUES (1, 'Product A', 100);
COMMIT;  -- At this point, guaranteed to persist

-- Even if system crashes 1 second later,
-- the order will still exist after restart
```

**Without Durability:** Committed data could be lost in crashes.

### Set Theory in SQL

SQL operations are based on **set theory**.

**Sets in SQL:**
- A table is a **set** of rows
- Rows are **elements** of the set
- Operations combine or filter sets

**Set Operations:**

```
Set A = {1, 2, 3, 4}
Set B = {3, 4, 5, 6}

UNION:        A ∪ B = {1, 2, 3, 4, 5, 6}
INTERSECT:    A ∩ B = {3, 4}
EXCEPT:       A - B = {1, 2}
```

**SQL Set Operations:**

```sql
-- UNION (combines, removes duplicates)
SELECT name FROM employees_2023
UNION
SELECT name FROM employees_2024;

-- UNION ALL (combines, keeps duplicates)
SELECT name FROM employees_2023
UNION ALL
SELECT name FROM employees_2024;

-- INTERSECT (common rows)
SELECT name FROM employees_2023
INTERSECT
SELECT name FROM employees_2024;

-- EXCEPT/MINUS (difference)
SELECT name FROM employees_2023
EXCEPT
SELECT name FROM employees_2024;
```

### Relational Algebra Basics

**Relational algebra** is the theoretical foundation of SQL.

**Core Operations:**

**1. Selection (σ) - WHERE clause**
```
σ(salary > 50000)(Employees)
```
SQL: `SELECT * FROM Employees WHERE salary > 50000`

**2. Projection (π) - SELECT clause**
```
π(name, salary)(Employees)
```
SQL: `SELECT name, salary FROM Employees`

**3. Join (⋈) - JOIN clause**
```
Employees ⋈ Departments
```
SQL: `SELECT * FROM Employees JOIN Departments`

**4. Union (∪) - UNION**
**5. Intersection (∩) - INTERSECT**
**6. Difference (−) - EXCEPT**

### Data Integrity Concepts

**Types of Integrity:**

**1. Entity Integrity**
- Each row must be uniquely identifiable
- Enforced by PRIMARY KEY
- No NULL in primary key

**2. Referential Integrity**
- Foreign keys must reference existing primary keys
- Enforced by FOREIGN KEY constraints
- Prevents orphaned records

**3. Domain Integrity**
- Column values must be from valid domain
- Enforced by data types, CHECK constraints
- Example: Age between 0 and 150

**4. User-Defined Integrity**
- Business rules specific to application
- Enforced by triggers, stored procedures
- Example: Order total = sum of line items

**Integrity Enforcement:**

```sql
-- Entity Integrity
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,  -- Unique, not null
    name VARCHAR(100)
);

-- Referential Integrity
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

-- Domain Integrity
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    price DECIMAL(10,2) CHECK (price >= 0),
    stock INT CHECK (stock >= 0)
);

-- User-Defined Integrity (via trigger)
CREATE TRIGGER check_order_total
BEFORE INSERT ON orders
FOR EACH ROW
BEGIN
    -- Custom business rule validation
END;
```

### Query Processing Theory

**Query Execution Pipeline:**

**1. Parsing**
```
SQL Text → Parse Tree
Validates syntax, checks table/column existence
```

**2. Binding**
```
Parse Tree → Bound Tree
Resolves names, checks permissions
```

**3. Optimization**
```
Bound Tree → Execution Plan
Finds most efficient way to execute
- Uses statistics
- Evaluates different strategies
- Chooses lowest cost
```

**4. Execution**
```
Execution Plan → Results
Executes operations step by step
```

**Example Query Processing:**

```sql
SELECT e.name, d.dept_name
FROM employees e
JOIN departments d ON e.dept_id = d.dept_id
WHERE e.salary > 50000;
```

**Possible Execution Plans:**

**Plan A:**
1. Scan employees (filter salary > 50000)
2. Scan departments
3. Join filtered employees with departments

**Plan B:**
1. Scan departments
2. Scan employees (filter salary > 50000)
3. Join departments with filtered employees

**Plan C (with indexes):**
1. Index seek on employees (salary > 50000)
2. For each employee, index seek on departments
3. Nested loop join

**Optimizer chooses based on:**
- Table sizes
- Index availability
- Data distribution
- Available memory

### Indexing Theory

**What is an Index?**

An index is a **separate data structure** that maintains a sorted order of column values with pointers to actual rows.

**Book Analogy:**
- **Table:** Book content (pages)
- **Index:** Book index at back
- **Without index:** Read every page to find "database"
- **With index:** Look up "database" in index → Jump to page 47

**B-Tree Structure (Most Common):**

```
                    [50, 100]
                   /    |    \
              [25,40] [75,90] [125,150]
             /  |  \   /  |  \   /  |  \
         [10] [30] [45] [60] [80] [95] [110] [130] [160]
          ↓    ↓    ↓    ↓    ↓    ↓    ↓     ↓     ↓
        [Pointers to actual rows in table]
```

**Properties:**
- **Balanced:** All leaf nodes at same depth
- **Sorted:** Keys in order
- **Efficient:** Search in O(log n) time

**Example Search for value 75:**
```
1. Start at root [50, 100]
   75 > 50 and 75 < 100 → Go to middle child
2. Reach [75, 90]
   75 = 75 → Found! Follow pointer to row
```

**Why Indexes Speed Up Queries:**

**Without Index:**
- **Full Table Scan:** Read every row sequentially
- **Complexity:** O(n) - linear time
- **For 1M rows:** Scan 1M rows

**With Index:**
- **Index Seek:** Navigate B-Tree
- **Complexity:** O(log n) - logarithmic time
- **For 1M rows:** ~20 lookups (log₂ 1,000,000 ≈ 20)

**Cost of Indexes:**

**Benefits:**
- ✅ Faster SELECT (especially with WHERE)
- ✅ Faster JOIN operations
- ✅ Faster ORDER BY (if sorted by index)

**Drawbacks:**
- ❌ Slower INSERT (must update index)
- ❌ Slower UPDATE (must maintain index)
- ❌ Slower DELETE (must update index)
- ❌ Additional storage space
- ❌ Maintenance overhead

**Trade-off Example:**
```
Table with 1M rows, no index:
- SELECT: 1000ms (full scan)
- INSERT: 1ms

Same table with index on 'email':
- SELECT: 5ms (index seek)
- INSERT: 3ms (insert row + update index)

Trade-off: Slower writes for much faster reads
Decision: Good for read-heavy workloads
```

### Normalization Theory

**Purpose of Normalization:**
- Eliminate redundancy
- Prevent anomalies
- Ensure data integrity
- Optimize storage

**Data Anomalies Without Normalization:**

**1. Insertion Anomaly**
```
Cannot add data without other data existing

Example (denormalized):
| emp_id | emp_name | dept_id | dept_name | dept_location |
Cannot add a department without an employee
```

**2. Update Anomaly**
```
Same data in multiple places must all be updated

Example:
| 1 | Alice | 10 | Sales | NYC |
| 2 | Bob   | 10 | Sales | NYC |
| 3 | Carol | 10 | Sales | NYC |

Change department location: Must update 3 rows
Miss one → Data inconsistency
```

**3. Deletion Anomaly**
```
Deleting data causes loss of other data

Example:
Last employee in department deleted → Department info lost
```

**Normalization Process:**

**Unnormalized → 1NF → 2NF → 3NF → BCNF → 4NF → 5NF**

Most applications stop at 3NF (good balance)

**Functional Dependency:**

**Definition:** Column B is functionally dependent on column A if:
- Each value of A determines exactly one value of B
- Written as: A → B

**Examples:**
```
employee_id → name (Each ID has one name)
employee_id → salary (Each ID has one salary)
employee_id → (name, salary) (Each ID determines both)

But NOT:
name → employee_id (Multiple people can have same name)
```

**Normalization Rules (Detailed):**

**1NF: Atomic Values**
- No repeating groups
- No multi-valued attributes
- Each cell contains single value

**2NF: No Partial Dependencies**
- Must be in 1NF
- No non-key column depends on part of composite key
- Only applies to tables with composite primary keys

**3NF: No Transitive Dependencies**
- Must be in 2NF
- No non-key column depends on another non-key column
- All non-key columns depend only on primary key

**When to Denormalize:**

**Trade-offs:**

| Normalized | Denormalized |
|------------|--------------|
| Less redundancy | More redundancy |
| Smaller storage | Larger storage |
| Complex queries (joins) | Simple queries |
| Easier updates | Harder updates |
| Better for writes | Better for reads |

**Denormalize when:**
- Read-heavy workload
- Performance critical
- Join cost too high
- Data rarely changes

**Example:**
```sql
-- Normalized (3NF): 3 tables, requires joins
customers → orders → order_items

-- Denormalized: 1 table, no joins needed
orders_denormalized (includes customer_name, customer_email)
```

### Transaction Theory

**What is a Transaction?**

A **transaction** is a sequence of one or more SQL operations treated as a single unit of work.

**Transaction States:**

```
BEGIN
  ↓
ACTIVE (executing operations)
  ↓
  ├→ FAILED (error occurred)
  │    ↓
  │  ABORTED (rollback)
  │    ↓
  │  TERMINATED
  │
  └→ PARTIALLY COMMITTED (all operations done)
       ↓
     COMMITTED (changes permanent)
       ↓
     TERMINATED
```

**Concurrency Control:**

**Problem:** Multiple transactions running simultaneously can interfere.

**Solution:** Locking mechanisms

**Lock Types:**

**1. Shared Lock (S)**
- For reading
- Multiple transactions can hold shared lock
- Prevents modifications

**2. Exclusive Lock (X)**
- For writing
- Only one transaction can hold exclusive lock
- Prevents all other access

**Lock Compatibility:**

|           | Shared (S) | Exclusive (X) |
|-----------|------------|---------------|
| **Shared (S)** | ✅ Compatible | ❌ Incompatible |
| **Exclusive (X)** | ❌ Incompatible | ❌ Incompatible |

**Deadlock:**

**Definition:** Two or more transactions wait for each other to release locks.

**Example:**
```
Time  Transaction 1              Transaction 2
1     Lock A (exclusive)         
2                                Lock B (exclusive)
3     Wait for B... →            
4                                ← Wait for A...
5     DEADLOCK!
```

**Deadlock Resolution:**
- **Detection:** System detects cycle
- **Victim selection:** Choose transaction to abort
- **Rollback:** Abort victim transaction
- **Retry:** Application retries transaction

**Two-Phase Locking (2PL):**

**Ensures serializability (transactions appear sequential)**

**Two Phases:**

1. **Growing Phase:** Acquire locks, cannot release
2. **Shrinking Phase:** Release locks, cannot acquire

```
Transaction Timeline:
├─ Growing Phase ─┤├─ Shrinking Phase ─┤
Acquire locks...   Release locks...
```

**Example:**
```sql
BEGIN TRANSACTION;
  -- Growing phase
  SELECT * FROM accounts WHERE id = 1 FOR UPDATE;  -- Acquire lock
  SELECT * FROM accounts WHERE id = 2 FOR UPDATE;  -- Acquire lock
  
  -- Operations
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
  
COMMIT;  -- Shrinking phase: Release all locks
```

### Join Theory (Deep Dive)

**Mathematical Foundation:**

**Cartesian Product:**
```
A = {1, 2}
B = {x, y, z}

A × B = {(1,x), (1,y), (1,z), (2,x), (2,y), (2,z)}
Size: |A| × |B| = 2 × 3 = 6
```

**CROSS JOIN = Cartesian Product**
```sql
SELECT * FROM A CROSS JOIN B;
-- Returns all combinations: 2 × 3 = 6 rows
```

**Join Algorithms:**

**1. Nested Loop Join**
```
For each row in Table A:
    For each row in Table B:
        If join condition matches:
            Return combined row

Complexity: O(n × m)
Good for: Small tables
```

**2. Hash Join**
```
Phase 1 (Build): Hash smaller table
Phase 2 (Probe): For each row in larger table,
                  look up hash table

Complexity: O(n + m)
Good for: Large tables, equality joins
```

**3. Merge Join**
```
Prerequisite: Both tables sorted on join key

Scan both tables in parallel:
    If keys match: Return combined row
    If A's key < B's key: Advance A
    If B's key < A's key: Advance B

Complexity: O(n + m) + sort cost
Good for: Large sorted tables, range joins
```

**Join Selectivity:**

**Definition:** Fraction of rows that satisfy join condition.

**Formula:**
```
Selectivity = (Matching rows) / (Total rows in Cartesian product)
```

**Example:**
```
Employees: 1000 rows
Departments: 10 rows

Cartesian product: 1000 × 10 = 10,000 rows
Actual JOIN result: 1000 rows (each employee in one dept)

Selectivity: 1000 / 10,000 = 0.1 (10%)
```

**High selectivity** (closer to 1): Many matches
**Low selectivity** (closer to 0): Few matches

**Optimizer uses selectivity to choose join algorithm.**

### Subquery Theory

**Subquery Types by Return:**

**1. Scalar Subquery**
- Returns: Single value (1 row, 1 column)
- Used: Anywhere a single value expected
- Example: `WHERE salary > (SELECT AVG(salary) FROM employees)`

**2. Row Subquery**
- Returns: Single row (1 row, N columns)
- Used: Tuple comparison
- Example: `WHERE (dept, salary) = (SELECT dept, MAX(salary) FROM ...)`

**3. Table Subquery**
- Returns: Multiple rows
- Used: IN, EXISTS, FROM clauses
- Example: `WHERE id IN (SELECT emp_id FROM ...)`

**Subquery Execution:**

**Uncorrelated (Independent):**
```sql
SELECT name FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

Execution:
1. Execute subquery ONCE: AVG(salary) = 50000
2. Use result in outer query: WHERE salary > 50000
```

**Correlated (Dependent):**
```sql
SELECT e1.name FROM employees e1
WHERE e1.salary > (SELECT AVG(e2.salary) FROM employees e2
                   WHERE e2.dept = e1.dept);

Execution:
For EACH row in e1:
  1. Execute subquery with current e1.dept
  2. Compare e1.salary with result
  3. Include row if condition true

Much slower: Subquery executes N times
```

**Subquery Optimization:**

**Problem:** Correlated subqueries can be slow

**Solution:** Rewrite as JOIN or window function

**Example:**
```sql
-- Slow: Correlated subquery
SELECT e1.name
FROM employees e1
WHERE e1.salary > (SELECT AVG(e2.salary)
                   FROM employees e2
                   WHERE e2.dept = e1.dept);

-- Fast: Window function
SELECT name
FROM (
    SELECT name, salary,
           AVG(salary) OVER (PARTITION BY dept) AS dept_avg
    FROM employees
) subquery
WHERE salary > dept_avg;

Speed improvement: 50-100x faster
```

---

# Part 1: Fundamentals

## 1. SQL Basics

### What is SQL?
- **S**tructured **Q**uery **L**anguage
- Declarative language (specify WHAT, not HOW)
- Used for managing relational databases

### RDBMS Core Concepts
```
Database
  └── Schema
      └── Tables (Relations)
          ├── Columns (Attributes)
          └── Rows (Records/Tuples)
```

### ACID Properties
| Property | Meaning |
|----------|---------|
| **A**tomicity | All or nothing |
| **C**onsistency | Valid state always |
| **I**solation | No interference |
| **D**urability | Changes persist |

### Data Types (Common)
```sql
-- Numeric
INT, BIGINT, DECIMAL(10,2), FLOAT

-- Text
VARCHAR(100), TEXT, CHAR(10)

-- Date/Time
DATE, TIME, DATETIME, TIMESTAMP

-- Boolean
BOOLEAN (TRUE/FALSE)
```

---

## 2. SELECT & Filtering

### Basic SELECT Structure
```sql
SELECT column1, column2           -- What to retrieve
FROM table_name                   -- From where
WHERE condition                   -- Filter rows
ORDER BY column [ASC|DESC]        -- Sort results
LIMIT n;                          -- Limit rows
```

### SELECT Variations
```sql
-- Specific columns
SELECT name, salary FROM employees;

-- All columns
SELECT * FROM employees;

-- With alias
SELECT name AS employee_name FROM employees;

-- With calculations
SELECT salary * 12 AS annual_salary FROM employees;

-- Distinct values
SELECT DISTINCT department FROM employees;
```

### WHERE Clause Essentials
```sql
-- Single condition
WHERE salary > 50000

-- Multiple conditions (AND)
WHERE department = 'IT' AND salary > 50000

-- Multiple conditions (OR)
WHERE department = 'IT' OR department = 'Sales'

-- Combining with parentheses
WHERE (department = 'IT' OR department = 'Sales') 
  AND salary > 50000
```

### ORDER BY
```sql
-- Ascending (default)
ORDER BY salary ASC

-- Descending
ORDER BY salary DESC

-- Multiple columns
ORDER BY department ASC, salary DESC
```

### LIMIT / TOP
```sql
-- MySQL/PostgreSQL
LIMIT 10

-- With offset (pagination)
LIMIT 10 OFFSET 20

-- SQL Server
TOP 10
```

---

## 3. Operators

### Comparison Operators
| Operator | Meaning | Example |
|----------|---------|---------|
| `=` | Equal | `salary = 50000` |
| `!=` or `<>` | Not equal | `dept != 'Sales'` |
| `>` | Greater than | `salary > 50000` |
| `<` | Less than | `age < 30` |
| `>=` | Greater or equal | `salary >= 50000` |
| `<=` | Less or equal | `age <= 65` |

### Logical Operators
```sql
-- AND (all must be true)
WHERE salary > 50000 AND department = 'IT'

-- OR (any can be true)
WHERE department = 'IT' OR department = 'Sales'

-- NOT (negates condition)
WHERE NOT department = 'Sales'
```

### Special Operators

**BETWEEN** (inclusive range)
```sql
WHERE salary BETWEEN 40000 AND 60000
-- Same as: salary >= 40000 AND salary <= 60000
```

**IN** (match any in list)
```sql
WHERE department IN ('IT', 'Sales', 'HR')
-- Same as: department = 'IT' OR department = 'Sales' OR department = 'HR'
```

**LIKE** (pattern matching)
```sql
WHERE name LIKE 'J%'        -- Starts with J
WHERE name LIKE '%son'      -- Ends with son
WHERE name LIKE '%an%'      -- Contains an
WHERE name LIKE '_a%'       -- Second char is 'a'
WHERE email LIKE '%@gmail.com'
```

**NULL Handling**
```sql
-- Check for NULL
WHERE phone IS NULL

-- Check for NOT NULL
WHERE phone IS NOT NULL

-- Provide default
SELECT COALESCE(phone, 'No phone') AS contact

-- ❌ WRONG
WHERE phone = NULL    -- Always returns NULL

-- ✅ CORRECT
WHERE phone IS NULL
```

---

# Part 2: Data Retrieval

## 4. SQL Joins

### Theory: Join Fundamentals

**Why Joins Exist:**

**Normalization Principle:**
- Data split into multiple tables to avoid redundancy
- Joins reassemble data when needed

**Example:**
```
Instead of:
orders: order_id, customer_name, customer_email, product, amount
(customer info repeated for each order)

We use:
customers: customer_id, name, email
orders: order_id, customer_id, product, amount
(customer info stored once, referenced by ID)

Join them to get complete picture
```

**Join Condition:**

The ON clause specifies **how to match rows** between tables.

```
Matching Rule:
For each row in Table A:
    For each row in Table B:
        If A.key = B.key:
            Combine rows and include in result

This is the fundamental join algorithm
```

**Join Cardinality:**

**1:1 Join** (One-to-One)
```
TableA: 100 rows
TableB: 100 rows
Result: ~100 rows (each A matches one B)
```

**1:N Join** (One-to-Many)
```
TableA: 100 rows
TableB: 1000 rows (10 per A row)
Result: ~1000 rows (each A matches multiple B)
```

**M:N Join** (Many-to-Many)
```
TableA: 100 rows
TableB: 100 rows
Result: Up to 10,000 rows (Cartesian product)
Need: Junction table to properly handle
```

**Understanding Result Size:**

```
Result rows ≤ (Table A rows) × (Table B rows)

Actual size depends on:
- Join condition selectivity
- Data distribution
- Relationship type
```

### Join Types Summary

| Join Type | Formula | Returns |
|-----------|---------|---------|
| **INNER JOIN** | A ∩ B | Only matching records |
| **LEFT JOIN** | A + (A ∩ B) | All from left + matches |
| **RIGHT JOIN** | (A ∩ B) + B | All from right + matches |
| **FULL OUTER** | A ∪ B | All from both tables |
| **CROSS JOIN** | A × B | Every combination |
| **SELF JOIN** | A JOIN A | Table with itself |

### INNER JOIN
```sql
-- Only matching rows
SELECT c.name, o.product
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id;

-- Multiple tables
SELECT c.name, p.product_name, o.quantity
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
INNER JOIN products p ON o.product_id = p.product_id;
```

**Result:** Only rows with matches in both tables

### LEFT JOIN
```sql
-- All customers (with or without orders)
SELECT c.name, o.product
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;

-- Find customers with NO orders
SELECT c.name
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

**Result:** All left table + matching right (NULL if no match)

### RIGHT JOIN
```sql
-- All orders (with or without customer info)
SELECT c.name, o.product
FROM customers c
RIGHT JOIN orders o ON c.customer_id = o.customer_id;
```

**Note:** Can always be rewritten as LEFT JOIN by swapping tables

### FULL OUTER JOIN
```sql
-- All customers and all orders
SELECT c.name, o.product
FROM customers c
FULL OUTER JOIN orders o ON c.customer_id = o.customer_id;

-- Find all unmatched records
WHERE c.customer_id IS NULL OR o.order_id IS NULL;
```

**MySQL workaround:**
```sql
SELECT * FROM customers c LEFT JOIN orders o ON c.id = o.customer_id
UNION
SELECT * FROM customers c RIGHT JOIN orders o ON c.id = o.customer_id;
```

### CROSS JOIN
```sql
-- All combinations (Cartesian product)
SELECT s.size, c.color
FROM sizes s
CROSS JOIN colors c;

-- If sizes has 3 rows, colors has 4 rows
-- Result: 3 × 4 = 12 rows
```

### SELF JOIN
```sql
-- Employee-Manager hierarchy
SELECT e.name AS employee, m.name AS manager
FROM employees e
INNER JOIN employees m ON e.manager_id = m.emp_id;

-- Find colleagues (same manager)
SELECT e1.name, e2.name, m.name AS manager
FROM employees e1
JOIN employees e2 ON e1.manager_id = e2.manager_id
  AND e1.emp_id < e2.emp_id
JOIN employees m ON e1.manager_id = m.emp_id;
```

### Join Best Practices
✅ **DO:**
- Use explicit JOIN syntax
- Always specify ON condition
- Use table aliases
- Index foreign keys

❌ **DON'T:**
- Use implicit joins (`FROM table1, table2`)
- Forget ON clause (creates CROSS JOIN)
- Use NATURAL JOIN in production
- Use RIGHT JOIN (use LEFT instead)

---

## 5. Aggregations & Grouping

### Theory: Aggregate Functions

**What are Aggregate Functions?**

Aggregate functions operate on **sets of rows** and return a **single value**.

**Mathematical Foundation:**

```
f: Set → Value

Examples:
COUNT({1, 2, 3, 4, 5}) = 5
SUM({1, 2, 3, 4, 5}) = 15
AVG({1, 2, 3, 4, 5}) = 3
MIN({1, 2, 3, 4, 5}) = 1
MAX({1, 2, 3, 4, 5}) = 5
```

**NULL Handling:**

**Critical Rule:** Most aggregate functions **ignore NULL** values.

```
Dataset: {10, 20, NULL, 30, NULL}

COUNT(*) = 5          (counts all rows)
COUNT(column) = 3     (counts non-NULL)
SUM(column) = 60      (ignores NULL)
AVG(column) = 20      (60/3, not 60/5)
```

**Why This Matters:**

```sql
-- Example: Average salary calculation
Salaries: {50000, 60000, NULL, 70000}

AVG(salary) = (50000 + 60000 + 70000) / 3 = 60000

NOT: (50000 + 60000 + 0 + 70000) / 4 = 45000
NULL ≠ 0
```

**GROUP BY Theory:**

**Conceptual Model:**

```
Original Table (ungrouped):
emp_id  dept      salary
1       Sales     50000
2       Sales     60000
3       IT        70000
4       IT        65000

GROUP BY dept creates partitions:

Partition 1 (Sales):        Partition 2 (IT):
emp_id  dept    salary      emp_id  dept    salary
1       Sales   50000       3       IT      70000
2       Sales   60000       4       IT      65000

Aggregate each partition:
dept     COUNT(*)  AVG(salary)
Sales    2         55000
IT       2         67500
```

**Why GROUP BY Rule Exists:**

```sql
-- ❌ WRONG - Ambiguous
SELECT name, department, COUNT(*)
FROM employees
GROUP BY department;

Problem: Which 'name' to show?
- Sales has: Alice, Bob
- Which one should appear in result?
- Ambiguous! Error!

-- ✅ CORRECT - Unambiguous
SELECT department, COUNT(*)
FROM employees
GROUP BY department;

Result: Each group gets exactly one row
- Sales: 2
- IT: 3
```

**Aggregate Functions

| Function | Purpose | Example Output |
|----------|---------|----------------|
| `COUNT(*)` | Count all rows | 100 |
| `COUNT(column)` | Count non-NULL | 95 |
| `SUM(column)` | Sum of values | 275000 |
| `AVG(column)` | Average | 55000.00 |
| `MIN(column)` | Minimum | 45000 |
| `MAX(column)` | Maximum | 65000 |

### Basic Aggregations
```sql
-- Total employees
SELECT COUNT(*) FROM employees;

-- Total salary expense
SELECT SUM(salary) FROM employees;

-- Average salary
SELECT AVG(salary) FROM employees;

-- Salary range
SELECT MIN(salary), MAX(salary) FROM employees;

-- Multiple aggregates
SELECT COUNT(*) AS emp_count,
       SUM(salary) AS total_sal,
       AVG(salary) AS avg_sal,
       MIN(salary) AS min_sal,
       MAX(salary) AS max_sal
FROM employees;
```

### GROUP BY Basics
```sql
-- Count employees per department
SELECT department, COUNT(*) AS emp_count
FROM employees
GROUP BY department;

-- Average salary per department
SELECT department, AVG(salary) AS avg_sal
FROM employees
GROUP BY department;

-- Multiple aggregates per group
SELECT department,
       COUNT(*) AS emp_count,
       SUM(salary) AS total_sal,
       AVG(salary) AS avg_sal
FROM employees
GROUP BY department;
```

### GROUP BY Rules
**Golden Rule:** Every column in SELECT must be either:
1. In the GROUP BY clause, OR
2. Inside an aggregate function

```sql
-- ❌ WRONG
SELECT name, department, COUNT(*)
FROM employees
GROUP BY department;  -- 'name' not grouped!

-- ✅ CORRECT
SELECT department, COUNT(*)
FROM employees
GROUP BY department;
```

### HAVING Clause
Filters **groups** (after aggregation)

```sql
-- Departments with more than 5 employees
SELECT department, COUNT(*) AS emp_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;

-- Departments with average salary > 50000
SELECT department, AVG(salary) AS avg_sal
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;
```

### WHERE vs HAVING

| Aspect | WHERE | HAVING |
|--------|-------|--------|
| **Filters** | Rows | Groups |
| **When** | Before GROUP BY | After GROUP BY |
| **Can use aggregates** | ❌ No | ✅ Yes |
| **Performance** | Better (filters early) | After calculations |

**Execution Order:**
```
FROM → WHERE → GROUP BY → Aggregate → HAVING → SELECT → ORDER BY
```

**Example:**
```sql
-- WHERE filters rows, HAVING filters groups
SELECT department, AVG(salary) AS avg_sal
FROM employees
WHERE salary > 40000              -- Filter rows first
GROUP BY department
HAVING AVG(salary) > 55000;       -- Filter groups after
```

### Common Patterns

**Top N per group:**
```sql
SELECT category, COUNT(*) AS product_count
FROM products
GROUP BY category
ORDER BY product_count DESC
LIMIT 5;
```

**Conditional aggregation:**
```sql
SELECT customer_id,
       COUNT(*) AS total_orders,
       SUM(CASE WHEN status = 'Shipped' THEN amount ELSE 0 END) AS shipped,
       SUM(CASE WHEN status = 'Pending' THEN amount ELSE 0 END) AS pending
FROM orders
GROUP BY customer_id;
```

**Percentage calculations:**
```sql
SELECT department,
       COUNT(*) AS dept_count,
       ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM employees), 2) AS pct
FROM employees
GROUP BY department;
```

---

# Part 3: Advanced Queries

## 6. Subqueries & CTEs

### Subquery Types

**1. Scalar Subquery** (returns single value)
```sql
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

**2. Row Subquery** (returns single row)
```sql
SELECT name
FROM employees
WHERE (department, salary) = (SELECT department, MAX(salary)
                               FROM employees
                               GROUP BY department
                               LIMIT 1);
```

**3. Table Subquery** (returns multiple rows)
```sql
SELECT name
FROM employees
WHERE department IN (SELECT dept_name FROM departments WHERE location = 'NYC');
```

### Subquery Locations

**In SELECT:**
```sql
SELECT name,
       salary,
       (SELECT AVG(salary) FROM employees) AS avg_salary
FROM employees;
```

**In FROM (Derived Table):**
```sql
SELECT dept, avg_sal
FROM (SELECT department AS dept, AVG(salary) AS avg_sal
      FROM employees
      GROUP BY department) AS dept_avg
WHERE avg_sal > 60000;
```

**In WHERE:**
```sql
-- With IN
SELECT name FROM employees
WHERE dept_id IN (SELECT dept_id FROM departments WHERE location = 'NYC');

-- With EXISTS
SELECT name FROM employees e
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.emp_id = e.id);

-- With comparison
SELECT name FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

### Correlated Subqueries
Depends on outer query (executes per row)

```sql
SELECT e1.name, e1.salary
FROM employees e1
WHERE e1.salary > (SELECT AVG(e2.salary)
                   FROM employees e2
                   WHERE e2.dept_id = e1.dept_id);
```

⚠️ **Performance:** Can be slow on large datasets

### CTEs (Common Table Expressions)

**Simple CTE:**
```sql
WITH high_earners AS (
    SELECT name, salary, dept_id
    FROM employees
    WHERE salary > 60000
)
SELECT * FROM high_earners
WHERE dept_id = 10;
```

**Multiple CTEs:**
```sql
WITH 
dept_avg AS (
    SELECT dept_id, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY dept_id
),
high_avg_depts AS (
    SELECT dept_id FROM dept_avg WHERE avg_sal > 60000
)
SELECT e.name, e.salary
FROM employees e
JOIN high_avg_depts h ON e.dept_id = h.dept_id;
```

**Recursive CTE:**
```sql
WITH RECURSIVE emp_hierarchy AS (
    -- Anchor: Top level
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive: Next level
    SELECT e.id, e.name, e.manager_id, eh.level + 1
    FROM employees e
    JOIN emp_hierarchy eh ON e.manager_id = eh.id
)
SELECT * FROM emp_hierarchy;
```

### CTE vs Subquery

| When to Use | CTE | Subquery |
|-------------|-----|----------|
| **Reused multiple times** | ✅ Better | ❌ Repeated |
| **One-time use** | ❌ Overhead | ✅ Better |
| **Recursive queries** | ✅ Required | ❌ Not supported |
| **Complex readability** | ✅ Better | ❌ Nested mess |

---

## 7. Window Functions

### Theory: What are Window Functions?

**Definition:** Window functions perform calculations across a set of rows related to the current row, WITHOUT collapsing rows like GROUP BY.

**Key Differences:**

| Aspect | GROUP BY | Window Function |
|--------|----------|----------------|
| **Result** | One row per group | All rows kept |
| **Aggregation** | Collapses rows | Adds columns |
| **Use case** | Summary reports | Row-level analytics |

**Conceptual Model:**

```
Think of looking through a "window" at related rows:

Without Window Function (GROUP BY):
dept     avg_salary
Sales    50000
IT       60000
(2 rows - collapsed)

With Window Function:
name     dept     salary   dept_avg_salary
Alice    Sales    45000    50000
Bob      Sales    55000    50000
Carol    IT       60000    60000
(3 rows - all kept, column added)
```

**Window Anatomy:**

```sql
FUNCTION() OVER (
    PARTITION BY column  -- Divide into groups (optional)
    ORDER BY column      -- Define row order (optional)
    ROWS BETWEEN ... AND ... -- Define frame (optional)
)
```

**How Window Functions Work:**

1. **Partition:** Divide rows into groups
2. **Order:** Sort rows within each partition
3. **Frame:** Define which rows to include in calculation
4. **Calculate:** Apply function to frame
5. **Return:** One value per row

### Ranking Functions

**Theory:** Assign a rank to each row based on ordering.

| Function | Behavior | Example Output |
|----------|----------|----------------|
| `ROW_NUMBER()` | Unique sequential | 1,2,3,4,5 |
| `RANK()` | Gaps for ties | 1,2,2,4,5 |
| `DENSE_RANK()` | No gaps | 1,2,2,3,4 |

```sql
SELECT name, salary, department,
       ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num,
       RANK() OVER (ORDER BY salary DESC) AS rank,
       DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;
```

### Offset Functions

**LEAD** (next row)
```sql
SELECT name, salary,
       LEAD(salary, 1) OVER (ORDER BY hire_date) AS next_salary
FROM employees;
```

**LAG** (previous row)
```sql
SELECT month, revenue,
       LAG(revenue, 1, 0) OVER (ORDER BY month) AS prev_revenue,
       revenue - LAG(revenue, 1, 0) OVER (ORDER BY month) AS growth
FROM monthly_sales;
```

### Aggregate Window Functions

**Running Total:**
```sql
SELECT date, amount,
       SUM(amount) OVER (ORDER BY date) AS running_total
FROM sales;
```

**Moving Average:**
```sql
SELECT date, sales,
       AVG(sales) OVER (
           ORDER BY date
           ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
       ) AS moving_avg_3day
FROM daily_sales;
```

### PARTITION BY
Divides data into groups (like GROUP BY but keeps all rows)

```sql
-- Rank within each department
SELECT name, department, salary,
       RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank
FROM employees;

-- Running total per department
SELECT department, date, amount,
       SUM(amount) OVER (PARTITION BY department ORDER BY date) AS dept_running_total
FROM sales;
```

### Frame Clauses

```sql
-- Current row and 2 preceding
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW

-- Current row and 1 following
ROWS BETWEEN CURRENT ROW AND 1 FOLLOWING

-- Entire partition
ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING

-- From start to current
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

### Window Function Template
```sql
SELECT columns,
       FUNCTION() OVER (
           [PARTITION BY column]
           [ORDER BY column]
           [ROWS BETWEEN ... AND ...]
       ) AS alias
FROM table;
```

### Common Use Cases

**Top N per group:**
```sql
WITH ranked AS (
    SELECT name, department, salary,
           ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rn
    FROM employees
)
SELECT * FROM ranked WHERE rn <= 3;
```

**Running total:**
```sql
SELECT date, amount,
       SUM(amount) OVER (ORDER BY date) AS cumulative
FROM orders;
```

**Previous/Next comparison:**
```sql
SELECT month, sales,
       LAG(sales) OVER (ORDER BY month) AS prev_month,
       sales - LAG(sales) OVER (ORDER BY month) AS change
FROM monthly_sales;
```

**Deduplication:**
```sql
WITH numbered AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY email ORDER BY created_at DESC) AS rn
    FROM users
)
SELECT * FROM numbered WHERE rn = 1;
```

---

# Part 4: Data Manipulation

## 8. DML Operations

### INSERT

**Single row:**
```sql
INSERT INTO employees (id, name, department, salary)
VALUES (1, 'Alice', 'IT', 60000);
```

**Multiple rows:**
```sql
INSERT INTO employees (id, name, department, salary)
VALUES 
    (1, 'Alice', 'IT', 60000),
    (2, 'Bob', 'Sales', 55000),
    (3, 'Carol', 'HR', 50000);
```

**From SELECT:**
```sql
INSERT INTO archive_employees (id, name, department)
SELECT id, name, department
FROM employees
WHERE hire_date < '2020-01-01';
```

**With defaults:**
```sql
CREATE TABLE orders (
    id INT PRIMARY KEY,
    status VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO orders (id) VALUES (1);  -- Uses defaults
```

### UPDATE

**Basic update:**
```sql
UPDATE employees
SET salary = 65000
WHERE id = 1;
```

**Multiple columns:**
```sql
UPDATE employees
SET salary = 70000,
    department = 'Senior IT'
WHERE id = 2;
```

**With calculation:**
```sql
UPDATE employees
SET salary = salary * 1.10
WHERE department = 'Sales';
```

**With JOIN:**
```sql
UPDATE employees e
SET e.bonus = d.bonus_rate * e.salary
FROM departments d
WHERE e.dept_id = d.id;
```

### DELETE

**With condition:**
```sql
DELETE FROM employees
WHERE department = 'Temp';
```

**All rows:**
```sql
DELETE FROM temp_data;
```

**With JOIN:**
```sql
DELETE e
FROM employees e
JOIN departments d ON e.dept_id = d.id
WHERE d.status = 'closed';
```

### MERGE / UPSERT

**Standard MERGE:**
```sql
MERGE INTO inventory AS target
USING new_stock AS source
ON target.product_id = source.product_id
WHEN MATCHED THEN
    UPDATE SET target.quantity = target.quantity + source.quantity
WHEN NOT MATCHED THEN
    INSERT (product_id, quantity) VALUES (source.product_id, source.quantity);
```

**PostgreSQL (ON CONFLICT):**
```sql
INSERT INTO users (id, username, login_count)
VALUES (1, 'alice', 1)
ON CONFLICT (id)
DO UPDATE SET login_count = users.login_count + 1;
```

**MySQL (REPLACE):**
```sql
REPLACE INTO settings (key_name, value)
VALUES ('theme', 'dark');
```

---

## 9. Transactions

### ACID Principles

| Property | Meaning | Example |
|----------|---------|---------|
| **Atomicity** | All or nothing | Both debit and credit succeed, or neither |
| **Consistency** | Valid state | Balance never negative |
| **Isolation** | No interference | Transactions don't see partial states |
| **Durability** | Survives crash | After COMMIT, data persists |

### Transaction Control

**Basic transaction:**
```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;
```

**With rollback:**
```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- Error occurs!
ROLLBACK;  -- Undo all changes
```

**With savepoint:**
```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
SAVEPOINT after_debit;

UPDATE accounts SET balance = balance + 100 WHERE id = 2;
-- Error in second update!

ROLLBACK TO after_debit;  -- Undo only second update
COMMIT;  -- Commit first update
```

### Best Practices
✅ Keep transactions short
✅ Handle errors properly
✅ Use appropriate isolation level
✅ Avoid user interaction in transactions

❌ Don't hold locks too long
❌ Don't nest transactions unnecessarily
❌ Don't ignore error handling

---

# Part 5: Database Design

## 10. Keys & Constraints

### Primary Key
- Uniquely identifies each row
- Cannot be NULL
- Only one per table

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100)
);

-- Composite primary key
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    PRIMARY KEY (student_id, course_id)
);
```

### Foreign Key
- References primary key in another table
- Enforces referential integrity

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

-- With cascade
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);
```

### Constraints

**UNIQUE:**
```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE,
    username VARCHAR(50) UNIQUE
);
```

**NOT NULL:**
```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL
);
```

**CHECK:**
```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    price DECIMAL(10,2) CHECK (price > 0),
    stock INT CHECK (stock >= 0)
);
```

**DEFAULT:**
```sql
CREATE TABLE tasks (
    task_id INT PRIMARY KEY,
    status VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 11. Normalization

### Normal Forms Summary

| Form | Rule | Eliminates |
|------|------|------------|
| **1NF** | Atomic values, unique rows | Repeating groups |
| **2NF** | 1NF + no partial dependencies | Partial dependencies |
| **3NF** | 2NF + no transitive dependencies | Transitive dependencies |

### 1NF (First Normal Form)
**Rule:** Each cell contains atomic (single) value

❌ **Violates 1NF:**
```
| student_id | courses                  |
|------------|--------------------------|
| 1          | Math, Physics, Chemistry |
```

✅ **In 1NF:**
```
| student_id | course    |
|------------|-----------|
| 1          | Math      |
| 1          | Physics   |
| 1          | Chemistry |
```

### 2NF (Second Normal Form)
**Rule:** 1NF + No partial dependencies (non-key columns depend on entire primary key)

❌ **Violates 2NF:**
```
Primary Key: (student_id, course_id)

| student_id | course_id | student_name | instructor |
|------------|-----------|--------------|------------|
| 1          | 101       | Alice        | Dr. Smith  |
```
*student_name depends only on student_id, not full key*

✅ **In 2NF:**
```
students: (student_id, student_name)
courses: (course_id, instructor)
enrollments: (student_id, course_id)
```

### 3NF (Third Normal Form)
**Rule:** 2NF + No transitive dependencies (non-key columns depend only on primary key)

❌ **Violates 3NF:**
```
| employee_id | name  | dept_id | dept_name |
|-------------|-------|---------|-----------|
| 1           | Alice | 10      | Sales     |
```
*dept_name depends on dept_id, not employee_id directly*

✅ **In 3NF:**
```
employees: (employee_id, name, dept_id)
departments: (dept_id, dept_name)
```

---

## 12. Relationships

### One-to-One (1:1)
```sql
-- Person → Passport
users: (user_id, username)
user_profiles: (user_id PK FK, bio, avatar)
```

### One-to-Many (1:M)
```sql
-- Customer → Orders
customers: (customer_id, name)
orders: (order_id, customer_id FK, date)
```

### Many-to-Many (M:N)
```sql
-- Students ↔ Courses (via junction table)
students: (student_id, name)
courses: (course_id, title)
enrollments: (student_id FK, course_id FK, PK(student_id, course_id))
```

---

# Part 6: Performance

## 13. Indexes & Optimization

### Theory: Performance Fundamentals

**Query Performance Factors:**

1. **Data Volume:** More rows = slower queries
2. **Disk I/O:** Reading from disk is slow
3. **CPU:** Processing data takes time
4. **Memory:** More memory = more caching
5. **Network:** Transferring data takes time

**Performance Metrics:**

```
Response Time = Queue Time + Service Time

Where:
- Queue Time: Waiting for resources
- Service Time: Actual processing time

Goal: Minimize both
```

**Bottleneck Analysis:**

```
Query Performance Bottlenecks:
1. Full table scans (no indexes)
2. Too many joins (complex queries)
3. Large result sets (no LIMIT)
4. Unoptimized queries (poor SQL)
5. Missing indexes
6. Wrong indexes (unused)
7. Locks and contention
```

**Cost-Based Optimization:**

**How Optimizer Works:**

1. **Parse query:** Validate syntax
2. **Generate plans:** Create multiple execution strategies
3. **Estimate costs:** Calculate CPU, I/O, memory for each plan
4. **Choose best:** Select lowest-cost plan
5. **Execute:** Run chosen plan

**Cost Factors:**
```
Cost = (CPU Cost) + (I/O Cost) + (Network Cost)

Where:
- CPU Cost: Operations performed
- I/O Cost: Disk reads/writes
- Network Cost: Data transfer
```

**Statistics:**

Database maintains statistics for optimization:
- **Cardinality:** Number of distinct values
- **Distribution:** How values spread
- **Density:** Frequency of values
- **Table size:** Number of rows

**Example:**
```sql
-- Optimizer uses statistics to decide:
SELECT * FROM employees WHERE department = 'IT';

Statistics show:
- employees table: 1,000,000 rows
- department column: 5 distinct values
- 'IT' department: 200,000 rows (20%)

Optimizer decides:
- Full table scan: Read 1M rows
- Index scan: Read 200K rows via index
- Choice: Index scan (4x fewer rows)
```

### Indexing Theory

**Index Types (Deep Dive):**

**1. Clustered Index**
- Table data physically sorted by index
- One per table (because data can only be in one order)
- Primary key automatically creates clustered index
- Fast for range queries

**2. Non-Clustered Index**
- Separate structure from table
- Points to table rows
- Can have multiple per table
- Extra lookup needed (index → table)

**3. Covering Index**
- Index contains all columns needed by query
- No table lookup required
- Fastest possible

**4. Composite Index**
- Index on multiple columns
- Order matters: (A, B, C)
  - Can use for: A, (A,B), (A,B,C)
  - Cannot use for: B, C, (B,C)

**Index Selection Strategy:**

```
Decision Tree:

Is column in WHERE/JOIN?
├─ Yes: Consider indexing
│  ├─ High cardinality? (many unique values)
│  │  ├─ Yes: Good candidate
│  │  └─ No: Skip (low cardinality)
│  ├─ Frequently queried?
│  │  ├─ Yes: Definitely index
│  │  └─ No: Maybe skip
│  └─ Large table?
│     ├─ Yes: Index important
│     └─ No: Table scan OK
└─ No: Don't index
```

**Index Overhead:**

**Write Operations:**
```
INSERT without index:
1. Add row to table
Time: O(1)

INSERT with index:
1. Add row to table
2. Update index (find position, insert)
Time: O(log n)

Trade-off: 3-5x slower writes for 100-1000x faster reads
```

**Storage:**
```
Index Size ≈ (Key Size + Pointer Size) × Number of Rows

Example:
- 1M rows
- Index on INT column (4 bytes)
- Pointer (8 bytes)
- Index size: (4 + 8) × 1M = 12 MB

Multiple indexes = Multiple × 12 MB
```

### B-Tree Deep Dive

**Why B-Tree?**

**Comparison:**

| Structure | Search | Insert | Sorted | Disk-Friendly |
|-----------|--------|--------|--------|---------------|
| Array | O(n) | O(1) | No | Yes |
| Hash Table | O(1) | O(1) | No | No |
| Binary Tree | O(log n) | O(log n) | Yes | No |
| **B-Tree** | **O(log n)** | **O(log n)** | **Yes** | **Yes** |

**B-Tree Properties:**

1. **Balanced:** All leaves at same level
2. **Multi-way:** Each node has multiple children
3. **Sorted:** Keys in order
4. **Disk-optimized:** Node size = disk page size

**B-Tree Parameters:**

```
Order = 3 (max 3 keys per node)

Node Structure:
[Key1 | Key2 | Key3]
  ↓     ↓     ↓    ↓
 Ptr0  Ptr1  Ptr2  Ptr3

Rules:
- Ptr0: Values < Key1
- Ptr1: Key1 < Values < Key2
- Ptr2: Key2 < Values < Key3
- Ptr3: Values > Key3
```

**Search Algorithm:**

```
Function Search(value, node):
    If node is leaf:
        If value in node.keys:
            Return pointer
        Else:
            Return NOT_FOUND
    
    For i = 0 to node.key_count:
        If value < node.keys[i]:
            Return Search(value, node.children[i])
    
    Return Search(value, node.children[node.key_count])

Time Complexity: O(log_b n)
where b = branching factor
```

**Insert Algorithm:**

```
Function Insert(value, node):
    If node is leaf:
        Insert value in sorted position
        If node is full:
            Split node
            Promote middle key to parent
    Else:
        Find appropriate child
        Insert(value, child)
        If child split:
            Add promoted key to node
            If node is full:
                Split node
```

**Real-World Example:**

```
Index on employee_id (INT, 4 bytes)
Disk page size: 4KB

Keys per node ≈ 4096 / 4 = 1024 keys

For 1 billion rows:
Tree height ≈ log₁₀₂₄(1,000,000,000) ≈ 3 levels

Search: 3 disk reads instead of 1,000,000,000!
```

### Query Optimization Theory

**Optimization Phases:**

**1. Logical Optimization**
- Simplify expressions
- Remove redundant operations
- Apply algebraic rules

**Example:**
```sql
-- Original
WHERE x = 10 AND x = 10

-- Optimized
WHERE x = 10

-- Original
WHERE NOT (x > 10)

-- Optimized
WHERE x <= 10
```

**2. Physical Optimization**
- Choose access methods
- Select join algorithms
- Determine join order

**Join Order Optimization:**

**Problem:** N tables → (N-1)! possible join orders

Example:
- 3 tables: 2 orders
- 4 tables: 6 orders
- 5 tables: 24 orders
- 10 tables: 362,880 orders!

**Solution:** Cost-based pruning

```
For each join order:
    Estimate cost
    If cost > best_so_far:
        Prune this branch
    Else:
        Continue exploring

Use heuristics:
- Small tables first
- Restrictive conditions early
- Cartesian products last
```

**Predicate Pushdown:**

**Theory:** Move filters as early as possible

```sql
-- Before optimization
SELECT *
FROM (
    SELECT * FROM large_table
) subquery
WHERE column = 'value';

-- After optimization (predicate pushed down)
SELECT *
FROM (
    SELECT * FROM large_table
    WHERE column = 'value'  -- Filter early!
) subquery;

Benefit: Filter 1M rows → 1K rows before processing
```

**Selectivity Estimation:**

```
Selectivity = (Matching rows) / (Total rows)

Low selectivity (0.01) = 1%: Use index
High selectivity (0.5) = 50%: Table scan better

Threshold typically: 15-20%
```

**Cardinality Estimation:**

```
Cardinality = Expected number of rows

Used to estimate:
- Result size
- Memory requirements
- Join costs

Example:
SELECT * FROM orders WHERE customer_id = 123;

Cardinality estimate:
- Total orders: 1M
- Distinct customers: 10K
- Estimate: 1M / 10K = 100 rows
```

### Execution Plans

**Plan Components:**

```
Execution Plan:
├─ Operations (what to do)
├─ Cost estimates (how expensive)
├─ Cardinality (how many rows)
└─ Access methods (how to get data)
```

**Operation Types:**

1. **Scan Operations**
   - Table Scan: Read entire table
   - Index Scan: Read index entries
   - Index Seek: Find specific entries

2. **Join Operations**
   - Nested Loop: Iterate combinations
   - Hash Join: Hash-based matching
   - Merge Join: Merge sorted inputs

3. **Aggregate Operations**
   - Stream Aggregate: On sorted data
   - Hash Aggregate: Hash-based grouping

4. **Sort Operations**
   - Sort: Order rows
   - Top: Get top N

**Reading EXPLAIN Output:**

```
Seq Scan on employees  (cost=0.00..35.50 rows=10 width=100)
  Filter: (department_id = 10)

Breakdown:
- Seq Scan: Sequential (full table) scan
- cost=0.00..35.50: Startup cost..total cost
- rows=10: Estimated rows returned
- width=100: Average row size (bytes)
- Filter: Condition applied
```

**Cost Interpretation:**

```
cost=0.00..35.50

0.00 = Startup cost (time to first row)
35.50 = Total cost (time to all rows)

Lower is better
Costs are relative, not absolute time
```

**Common Patterns:**

**Good:**
```
Index Seek → Nested Loop → Output
(Fast: Uses index, small dataset)
```

**Bad:**
```
Table Scan → Hash Join → Table Scan → Output
(Slow: Full scans, large dataset)
```

## 14. Query Optimization

### B-Tree Index Basics
```
Balanced tree structure for fast lookups
Search: O(log n) instead of O(n)

         [50]
        /    \
    [25]      [75]
    /  \      /  \
 [10] [40] [60] [90]
```

### When to Index
✅ **DO index:**
- Primary keys (automatic)
- Foreign keys
- Columns in WHERE clauses
- Columns in JOIN conditions
- Columns in ORDER BY
- High cardinality columns

❌ **DON'T index:**
- Small tables (<1000 rows)
- Low cardinality columns (gender, boolean)
- Frequently updated columns
- Wide columns (TEXT, BLOB)

### Index Types

**Single column:**
```sql
CREATE INDEX idx_email ON users(email);
```

**Composite index:**
```sql
CREATE INDEX idx_name ON users(last_name, first_name);

-- Uses index
WHERE last_name = 'Smith'
WHERE last_name = 'Smith' AND first_name = 'John'

-- Doesn't use index
WHERE first_name = 'John'  -- Skips leftmost column
```

**Covering index:**
```sql
CREATE INDEX idx_user_details ON users(email, name, city);

-- All columns in index, no table lookup needed
SELECT email, name, city FROM users WHERE email = 'alice@example.com';
```

### EXPLAIN Usage
```sql
EXPLAIN SELECT * FROM employees WHERE department = 'IT';

-- Key indicators in output:
-- Type: ALL (bad - full scan), index (good)
-- Key: index name being used
-- Rows: estimated rows scanned
-- Extra: Using index, Using where, etc.
```

---

## 14. Query Optimization

### Optimization Techniques

**1. Select only needed columns:**
```sql
-- ❌ Bad
SELECT * FROM employees;

-- ✅ Good
SELECT employee_id, name, salary FROM employees;
```

**2. Avoid functions on indexed columns:**
```sql
-- ❌ Bad (index not used)
WHERE YEAR(date) = 2024

-- ✅ Good (index used)
WHERE date >= '2024-01-01' AND date < '2025-01-01'
```

**3. Use EXISTS instead of IN for large datasets:**
```sql
-- ❌ Slower
SELECT name FROM customers
WHERE customer_id IN (SELECT customer_id FROM orders);

-- ✅ Faster
SELECT name FROM customers c
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id);
```

**4. Avoid correlated subqueries:**
```sql
-- ❌ Bad (runs for each row)
SELECT name,
       (SELECT AVG(salary) FROM employees e2 WHERE e2.dept_id = e1.dept_id)
FROM employees e1;

-- ✅ Good (runs once)
SELECT name, AVG(salary) OVER (PARTITION BY dept_id)
FROM employees;
```

**5. Filter early with WHERE:**
```sql
-- ✅ Good
SELECT department, COUNT(*)
FROM employees
WHERE salary > 50000
GROUP BY department;
```

### Common Anti-Patterns

**N+1 Query Problem:**
```python
# ❌ Bad: 1 + N queries
orders = db.query("SELECT * FROM orders")
for order in orders:
    customer = db.query(f"SELECT * FROM customers WHERE id = {order.customer_id}")

# ✅ Good: 1 query with JOIN
results = db.query("""
    SELECT o.*, c.name
    FROM orders o
    JOIN customers c ON o.customer_id = c.id
""")
```

**SELECT * Anti-pattern:**
- Fetches unnecessary data
- Prevents covering indexes
- Breaks when schema changes

**OR in WHERE:**
```sql
-- ❌ May not use index
WHERE category = 'A' OR price < 100

-- ✅ Better
WHERE category = 'A'
UNION
WHERE price < 100
```

### Performance Checklist
- [ ] Index foreign keys
- [ ] Use appropriate JOIN type
- [ ] Avoid SELECT *
- [ ] Filter with WHERE before JOIN
- [ ] Use window functions instead of correlated subqueries
- [ ] Test with production-like data volumes
- [ ] Monitor slow query logs
- [ ] Use EXPLAIN to verify index usage

---

# Part 7: Interview Patterns

## 15. Common Interview Questions

### Second Highest Salary
```sql
-- Method 1: LIMIT with OFFSET
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;

-- Method 2: Subquery with MAX
SELECT MAX(salary)
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);

-- Method 3: DENSE_RANK
SELECT salary
FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rank
    FROM employees
) ranked
WHERE rank = 2;
```

### Find Duplicates
```sql
-- Find duplicate emails
SELECT email, COUNT(*) AS duplicate_count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

-- Show all duplicate records
SELECT *
FROM (
    SELECT *, COUNT(*) OVER (PARTITION BY email) AS cnt
    FROM users
) subquery
WHERE cnt > 1;
```

### Remove Duplicates
```sql
-- Keep only one copy (lowest ID)
DELETE FROM users
WHERE user_id NOT IN (
    SELECT MIN(user_id)
    FROM users
    GROUP BY email
);

-- Or using ROW_NUMBER
DELETE FROM users
WHERE user_id IN (
    SELECT user_id
    FROM (
        SELECT user_id,
               ROW_NUMBER() OVER (PARTITION BY email ORDER BY user_id) AS rn
        FROM users
    ) ranked
    WHERE rn > 1
);
```

### Top N per Group
```sql
-- Top 3 employees per department by salary
WITH ranked AS (
    SELECT name, department, salary,
           ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rn
    FROM employees
)
SELECT name, department, salary
FROM ranked
WHERE rn <= 3;
```

---

## 16. Real-World Patterns

### Gaps in Dates
```sql
-- Find missing dates in sequence
WITH RECURSIVE date_range AS (
    SELECT MIN(date) AS date FROM attendance
    UNION ALL
    SELECT date + INTERVAL '1 day'
    FROM date_range
    WHERE date < (SELECT MAX(date) FROM attendance)
)
SELECT date AS missing_date
FROM date_range
WHERE date NOT IN (SELECT date FROM attendance);
```

### Consecutive Records
```sql
-- Find login streaks
WITH grouped AS (
    SELECT user_id, login_date,
           login_date - ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) * INTERVAL '1 day' AS grp
    FROM logins
)
SELECT user_id,
       MIN(login_date) AS streak_start,
       MAX(login_date) AS streak_end,
       COUNT(*) AS days
FROM grouped
GROUP BY user_id, grp
HAVING COUNT(*) >= 3;
```

### Running Totals
```sql
SELECT date, amount,
       SUM(amount) OVER (ORDER BY date) AS running_total
FROM sales;
```

### Pivot (Rows to Columns)
```sql
-- Monthly sales by product
SELECT product,
       SUM(CASE WHEN month = 'Jan' THEN sales ELSE 0 END) AS Jan,
       SUM(CASE WHEN month = 'Feb' THEN sales ELSE 0 END) AS Feb,
       SUM(CASE WHEN month = 'Mar' THEN sales ELSE 0 END) AS Mar
FROM monthly_sales
GROUP BY product;
```

### Year-over-Year Growth
```sql
SELECT year,
       sales,
       LAG(sales) OVER (ORDER BY year) AS prev_year_sales,
       ROUND((sales - LAG(sales) OVER (ORDER BY year)) * 100.0 / 
             LAG(sales) OVER (ORDER BY year), 2) AS yoy_growth_pct
FROM yearly_sales;
```

### Active Users (Complex Logic)
```sql
WITH activity AS (
    SELECT u.user_id,
           MAX(CASE WHEN l.login_date >= CURRENT_DATE - 7 THEN 1 ELSE 0 END) AS recent_login,
           MAX(CASE WHEN p.purchase_date >= CURRENT_DATE - 30 THEN 1 ELSE 0 END) AS recent_purchase
    FROM users u
    LEFT JOIN logins l ON u.user_id = l.user_id
    LEFT JOIN purchases p ON u.user_id = p.user_id
    GROUP BY u.user_id
)
SELECT user_id,
       CASE
           WHEN recent_login = 1 AND recent_purchase = 1 THEN 'Active'
           WHEN recent_login = 1 THEN 'Engaged'
           WHEN recent_purchase = 1 THEN 'Buyer'
           ELSE 'Inactive'
       END AS status
FROM activity;
```

---

# Quick Reference Tables

## SQL Execution Order
```
1. FROM
2. WHERE
3. GROUP BY
4. Aggregate Functions (COUNT, SUM, etc.)
5. HAVING
6. SELECT
7. DISTINCT
8. ORDER BY
9. LIMIT/TOP
```

## Join Decision Matrix
| Need | Use |
|------|-----|
| Only matching records | INNER JOIN |
| All from left + matches | LEFT JOIN |
| All from right + matches | RIGHT JOIN |
| All from both | FULL OUTER JOIN |
| All combinations | CROSS JOIN |
| Same table hierarchy | SELF JOIN |

## When to Use What

### Subquery vs CTE
| Situation | Use |
|-----------|-----|
| Reused multiple times | CTE |
| One-time, simple | Subquery |
| Recursive logic | CTE (required) |
| Performance critical | Test both |

### JOIN vs EXISTS
| Need | Use |
|------|-----|
| Columns from both tables | JOIN |
| Just checking existence | EXISTS |
| 1:Many with duplicates | EXISTS |
| Aggregate from related | JOIN |

### Window Function vs Subquery
| Need | Use |
|------|-----|
| Ranking | Window Function |
| Running totals | Window Function |
| Per-row calculations | Window Function |
| Simple aggregate | Either |

## Performance Quick Tips
1. **Index foreign keys** - 10-100x speedup
2. **Use EXISTS over IN** - 2-10x for large datasets
3. **Window functions over correlated subqueries** - 50x speedup
4. **Filter early with WHERE** - Reduces data before processing
5. **Select specific columns** - 2-10x less data transfer
6. **Avoid functions on indexed columns** - Prevents index use

## Common Mistakes Checklist
- [ ] Using `= NULL` instead of `IS NULL`
- [ ] Forgetting `ON` clause in JOIN (creates CROSS JOIN)
- [ ] Mixing aggregated and non-aggregated columns without GROUP BY
- [ ] Using aggregate functions in WHERE (use HAVING)
- [ ] Not handling NULL in calculations
- [ ] Using SELECT * in production
- [ ] Forgetting ORDER BY with LIMIT (unpredictable results)
- [ ] Using RIGHT JOIN instead of LEFT JOIN
- [ ] Not testing with production data volumes
- [ ] Ignoring EXPLAIN output

---

# Interview Preparation Checklist

## Must-Know Concepts
- [ ] JOINS (especially INNER, LEFT)
- [ ] GROUP BY and HAVING
- [ ] Window Functions (ROW_NUMBER, RANK, SUM OVER)
- [ ] CTEs for complex queries
- [ ] Subqueries (all three locations)
- [ ] Indexes and performance basics
- [ ] Primary/Foreign Keys
- [ ] Normalization (1NF, 2NF, 3NF)

## Must-Practice Patterns
- [ ] Second/Nth highest value
- [ ] Top N per group
- [ ] Finding duplicates
- [ ] Removing duplicates
- [ ] Running totals
- [ ] Gaps in sequences
- [ ] Consecutive records
- [ ] Pivot/Unpivot
- [ ] Year-over-year growth
- [ ] Active users (complex logic)

## Must-Understand Trade-offs
- [ ] JOIN vs EXISTS
- [ ] Subquery vs CTE
- [ ] WHERE vs HAVING
- [ ] Normalization vs Denormalization
- [ ] Indexing benefits vs costs
- [ ] Window Functions vs Subqueries

---

# Final Tips for Success

## During Interviews
1. **Clarify requirements** - Ask about edge cases, NULL handling
2. **Think aloud** - Explain your approach
3. **Start simple** - Get basic query working first
4. **Test mentally** - Walk through with sample data
5. **Optimize if asked** - Discuss indexes, alternatives

## Study Strategy
1. **Master fundamentals first** - SELECT, WHERE, JOINS
2. **Practice patterns daily** - One pattern per day
3. **Write queries by hand** - Don't rely on autocomplete
4. **Understand, don't memorize** - Know why, not just what
5. **Test on real databases** - Practice with actual data

## Common Interview Topics by Company Type
**Startups:** Practical SQL, quick problem-solving
**Big Tech:** Complex queries, optimization, scalability
**Data Teams:** Analytics patterns, window functions
**Finance:** Accuracy, transactions, ACID properties

---

**You're now equipped with comprehensive SQL knowledge!**
**Practice regularly, understand deeply, and you'll ace any SQL interview.** 🚀

---

*End of SQL Complete Revision Guide*
