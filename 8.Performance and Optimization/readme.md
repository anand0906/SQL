# SQL Performance & Optimization - Pro Level Guide

## Index
1. [Introduction](#introduction)
2. [Indexes](#indexes)
   - [What are Indexes](#what-are-indexes)
   - [B-Tree Index Basics](#b-tree-index-basics)
   - [Types of Indexes](#types-of-indexes)
   - [When to Use Indexes](#when-to-use-indexes)
   - [When NOT to Use Indexes](#when-not-to-use-indexes)
3. [Execution Plans (EXPLAIN)](#execution-plans-explain)
   - [Understanding EXPLAIN](#understanding-explain)
   - [Reading Execution Plans](#reading-execution-plans)
   - [Common Operations](#common-operations)
   - [Cost Analysis](#cost-analysis)
4. [Query Optimization](#query-optimization)
   - [SELECT Optimization](#select-optimization)
   - [JOIN Optimization](#join-optimization)
   - [WHERE Clause Optimization](#where-clause-optimization)
   - [Avoiding Function Calls on Indexed Columns](#avoiding-function-calls-on-indexed-columns)
5. [Common Anti-Patterns](#common-anti-patterns)
   - [SELECT * Anti-Pattern](#select--anti-pattern)
   - [N+1 Query Problem](#n1-query-problem)
   - [Unnecessary DISTINCT](#unnecessary-distinct)
   - [OR in WHERE Clause](#or-in-where-clause)
   - [Implicit Type Conversions](#implicit-type-conversions)
6. [When NOT to Use Subqueries](#when-not-to-use-subqueries)
   - [Correlated Subqueries in SELECT](#correlated-subqueries-in-select)
   - [Repeated Subqueries](#repeated-subqueries)
   - [Subqueries vs Window Functions](#subqueries-vs-window-functions)
7. [JOIN vs EXISTS](#join-vs-exists)
   - [Performance Comparison](#performance-comparison)
   - [When to Use JOIN](#when-to-use-join)
   - [When to Use EXISTS](#when-to-use-exists)
8. [CTE vs Subquery Performance](#cte-vs-subquery-performance)
   - [Materialization Differences](#materialization-differences)
   - [Optimization Behavior](#optimization-behavior)
   - [When to Use Each](#when-to-use-each)
9. [Best Practices Checklist](#best-practices-checklist)
10. [Advanced Interview Questions](#advanced-interview-questions)

---

## Introduction

**Performance Optimization** is the art of making queries run faster and use fewer resources.

**Key Metrics:**
- **Query execution time** - How long the query takes
- **Resource usage** - CPU, memory, I/O
- **Scalability** - Performance with growing data

**Optimization Principles:**
1. **Measure first** - Use EXPLAIN to understand current performance
2. **Index strategically** - Balance read vs write performance
3. **Write efficient queries** - Avoid unnecessary work
4. **Test with real data** - Small datasets hide problems

---

## Indexes

### What are Indexes

An **index** is a data structure that improves the speed of data retrieval operations at the cost of additional storage and slower writes.

**Analogy:**
- **Without index**: Like reading a book page by page to find a word
- **With index**: Like using a book's index to jump directly to the page

**Key Characteristics:**
- Speeds up SELECT queries with WHERE, JOIN, ORDER BY
- Slows down INSERT, UPDATE, DELETE (index must be updated)
- Takes additional storage space
- Automatically created for PRIMARY KEY and UNIQUE constraints

---

### B-Tree Index Basics

**B-Tree (Balanced Tree)** is the most common index type in databases.

**Structure:**
```
         [50]
        /    \
    [25]      [75]
    /  \      /  \
 [10] [40] [60] [90]
```

**How it Works:**
1. Data organized in a balanced tree structure
2. Each node contains keys and pointers
3. Search requires O(log n) operations
4. Tree stays balanced through node splits and merges

**Example Search for value 60:**
```
Start at root (50)
→ 60 > 50, go right
→ Reach (75)
→ 60 < 75, go left
→ Found 60
```

**Operations:** 3 steps instead of scanning all rows

---

### Types of Indexes

| Index Type | Description | Use Case |
|------------|-------------|----------|
| **Single Column** | Index on one column | WHERE id = 5 |
| **Composite** | Index on multiple columns | WHERE last_name='Smith' AND first_name='John' |
| **Unique** | Ensures uniqueness | Email, username |
| **Full-Text** | Text search | Article content search |
| **Covering** | Includes all query columns | Avoid table lookup |

**Example:**

```sql
-- Single column index
CREATE INDEX idx_email ON users(email);

-- Composite index (order matters!)
CREATE INDEX idx_name ON users(last_name, first_name);

-- Covering index
CREATE INDEX idx_user_details ON users(email, name, city);
```

**Composite Index Column Order:**

```sql
-- Index: (last_name, first_name)

-- ✅ Uses index (leftmost prefix)
SELECT * FROM users WHERE last_name = 'Smith';

-- ✅ Uses index (both columns)
SELECT * FROM users WHERE last_name = 'Smith' AND first_name = 'John';

-- ❌ Cannot use index (skips leftmost column)
SELECT * FROM users WHERE first_name = 'John';
```

---

### When to Use Indexes

**Create indexes on columns used in:**

1. **WHERE clauses** (filtering)
   ```sql
   SELECT * FROM orders WHERE customer_id = 123;
   -- Index on customer_id
   ```

2. **JOIN conditions**
   ```sql
   SELECT * FROM orders o JOIN customers c ON o.customer_id = c.id;
   -- Index on orders.customer_id
   ```

3. **ORDER BY clauses** (sorting)
   ```sql
   SELECT * FROM products ORDER BY price DESC;
   -- Index on price
   ```

4. **Foreign keys** (joins and referential integrity)
   ```sql
   -- Always index foreign keys
   CREATE INDEX idx_order_customer ON orders(customer_id);
   ```

5. **Columns with high cardinality** (many distinct values)
   ```sql
   -- Good: email (unique per user)
   -- Bad: gender (only M/F/Other)
   ```

---

### When NOT to Use Indexes

**Avoid indexes when:**

1. **Small tables** (<1000 rows)
   - Full table scan is faster than index lookup

2. **Low cardinality columns** (few distinct values)
   ```sql
   -- ❌ Bad index: only 2-3 values
   CREATE INDEX idx_gender ON users(gender);
   ```

3. **Columns frequently updated**
   - Each UPDATE requires index update

4. **Wide columns** (TEXT, BLOB)
   - Use full-text indexes instead

5. **Tables with heavy writes**
   - More indexes = slower INSERT/UPDATE/DELETE

**Example - When Index Hurts:**

**Scenario:** Table with 100 rows

```sql
-- Without index: Scan 100 rows
SELECT * FROM small_table WHERE status = 'active';
-- Time: 0.5ms

-- With index: Read index + lookup rows
CREATE INDEX idx_status ON small_table(status);
SELECT * FROM small_table WHERE status = 'active';
-- Time: 0.8ms (slower!)
```

---

## Execution Plans (EXPLAIN)

### Understanding EXPLAIN

**EXPLAIN** shows how the database will execute your query.

**Syntax:**
```sql
EXPLAIN SELECT * FROM users WHERE email = 'alice@example.com';

-- For more details (PostgreSQL)
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'alice@example.com';
```

**Why Use EXPLAIN:**
- Identify slow queries
- Verify index usage
- Understand query costs
- Find bottlenecks

---

### Reading Execution Plans

**Example Query:**

```sql
EXPLAIN SELECT * FROM employees WHERE department_id = 10;
```

**Output (PostgreSQL):**

```
Seq Scan on employees  (cost=0.00..35.50 rows=10 width=100)
  Filter: (department_id = 10)
```

**Key Components:**

| Component | Meaning |
|-----------|---------|
| **Seq Scan** | Sequential (full table) scan |
| **Index Scan** | Using an index |
| **cost=0.00..35.50** | Startup cost..total cost |
| **rows=10** | Estimated rows returned |
| **width=100** | Average row size in bytes |

**With Index:**

```sql
CREATE INDEX idx_dept ON employees(department_id);
EXPLAIN SELECT * FROM employees WHERE department_id = 10;
```

**Output:**

```
Index Scan using idx_dept on employees  (cost=0.29..8.31 rows=10 width=100)
  Index Cond: (department_id = 10)
```

**Improvement:** Cost reduced from 35.50 to 8.31!

---

### Common Operations

**Operations in Execution Plans:**

| Operation | Description | Performance |
|-----------|-------------|-------------|
| **Seq Scan** | Full table scan | Slow on large tables |
| **Index Scan** | Using index to find rows | Fast for selective queries |
| **Index Only Scan** | All data from index | Fastest (no table lookup) |
| **Bitmap Heap Scan** | Multiple index scans combined | Good for OR conditions |
| **Nested Loop** | JOIN method for small sets | Good for small result sets |
| **Hash Join** | JOIN method using hash table | Good for large result sets |
| **Merge Join** | JOIN on sorted data | Good for sorted/indexed data |

**Example - Index Only Scan:**

```sql
-- Covering index
CREATE INDEX idx_user_email_name ON users(email, name);

-- Query uses only indexed columns
EXPLAIN SELECT email, name FROM users WHERE email = 'alice@example.com';
```

**Output:**
```
Index Only Scan using idx_user_email_name on users
  Index Cond: (email = 'alice@example.com')
```

**No table lookup needed** - all data in index!

---

### Cost Analysis

**Understanding Costs:**

```sql
EXPLAIN SELECT * FROM large_table WHERE id = 100;
```

**Output:**
```
Index Scan using pk_id on large_table  (cost=0.42..8.44 rows=1 width=200)
  Index Cond: (id = 100)
```

**Cost Breakdown:**
- **0.42** = Startup cost (time to get first row)
- **8.44** = Total cost (time to get all rows)
- **Difference (8.02)** = Cost to fetch remaining rows

**Comparison:**

| Query Type | Startup Cost | Total Cost | Performance |
|------------|--------------|------------|-------------|
| Index lookup | 0.42 | 8.44 | Fast |
| Full scan | 0.00 | 1234.56 | Slow |
| Index + sort | 0.42 | 45.67 | Medium |

---

## Query Optimization

### SELECT Optimization

**❌ Bad: SELECT * (Fetches all columns)**

```sql
SELECT * FROM employees WHERE department_id = 10;
```

**Problems:**
- Transfers unnecessary data
- Cannot use covering indexes
- Slower over network

**✅ Good: Select only needed columns**

```sql
SELECT employee_id, name, salary FROM employees WHERE department_id = 10;
```

**Benefits:**
- Less data transfer
- Can use covering index
- Faster execution

**Performance Comparison:**

| Approach | Columns Retrieved | Data Size | Time |
|----------|-------------------|-----------|------|
| SELECT * | 20 columns | 500 KB | 150ms |
| SELECT specific | 3 columns | 75 KB | 45ms |

---

### JOIN Optimization

**❌ Bad: Multiple joins without indexes**

```sql
SELECT *
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN products p ON o.product_id = p.id
WHERE o.order_date > '2024-01-01';
```

**Without indexes:** Full table scans on all tables

**✅ Good: Indexed foreign keys**

```sql
-- Create indexes
CREATE INDEX idx_order_customer ON orders(customer_id);
CREATE INDEX idx_order_product ON orders(product_id);
CREATE INDEX idx_order_date ON orders(order_date);

-- Same query now uses indexes
SELECT *
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN products p ON o.product_id = p.id
WHERE o.order_date > '2024-01-01';
```

**Performance:**
- Without indexes: 2000ms
- With indexes: 50ms

---

### WHERE Clause Optimization

**❌ Bad: Function on indexed column**

```sql
-- Index on email, but function prevents its use
SELECT * FROM users WHERE UPPER(email) = 'ALICE@EXAMPLE.COM';
```

**✅ Good: Function on literal value**

```sql
-- Index can be used
SELECT * FROM users WHERE email = LOWER('ALICE@EXAMPLE.COM');
```

**Or use functional index:**

```sql
-- Create functional index
CREATE INDEX idx_email_upper ON users(UPPER(email));

-- Now this works
SELECT * FROM users WHERE UPPER(email) = 'ALICE@EXAMPLE.COM';
```

---

### Avoiding Function Calls on Indexed Columns

**Common Anti-Pattern:**

| ❌ Bad (Index Not Used) | ✅ Good (Index Used) |
|------------------------|---------------------|
| `WHERE YEAR(date) = 2024` | `WHERE date >= '2024-01-01' AND date < '2025-01-01'` |
| `WHERE salary * 1.1 > 50000` | `WHERE salary > 50000 / 1.1` |
| `WHERE SUBSTRING(name, 1, 1) = 'A'` | `WHERE name LIKE 'A%'` |
| `WHERE id + 10 = 100` | `WHERE id = 90` |

**Example:**

```sql
-- ❌ Bad: Function prevents index use
EXPLAIN SELECT * FROM orders WHERE YEAR(order_date) = 2024;
-- Result: Seq Scan (full table scan)

-- ✅ Good: Index can be used
EXPLAIN SELECT * FROM orders 
WHERE order_date >= '2024-01-01' 
  AND order_date < '2025-01-01';
-- Result: Index Scan
```

---

## Common Anti-Patterns

### SELECT * Anti-Pattern

**Problem:** Fetches unnecessary columns, prevents covering indexes.

**Example:**

```sql
-- ❌ Bad
SELECT * FROM users WHERE user_id = 123;

-- ✅ Good
SELECT user_id, username, email FROM users WHERE user_id = 123;
```

**Impact:**
- 10x more data transferred
- Cannot use covering index
- Breaks when table schema changes

---

### N+1 Query Problem

**Problem:** Executing N additional queries in a loop.

**❌ Bad Example:**

```python
# Get all orders
orders = execute("SELECT * FROM orders")

# For each order, get customer (N queries!)
for order in orders:
    customer = execute(f"SELECT * FROM customers WHERE id = {order.customer_id}")
    print(customer.name)
```

**Queries:** 1 + N (if 100 orders, 101 queries!)

**✅ Good Example:**

```python
# Single query with JOIN
results = execute("""
    SELECT o.*, c.name 
    FROM orders o
    JOIN customers c ON o.customer_id = c.id
""")

for row in results:
    print(row.name)
```

**Queries:** 1 only!

**Performance:**
- N+1 approach: 5000ms (101 queries)
- JOIN approach: 50ms (1 query)

---

### Unnecessary DISTINCT

**Problem:** DISTINCT is expensive - sorts all results to remove duplicates.

**❌ Bad: Using DISTINCT to fix bad query**

```sql
-- DISTINCT hides the real problem
SELECT DISTINCT c.customer_name
FROM customers c
JOIN orders o ON c.id = o.customer_id;
```

**✅ Good: Fix the root cause**

```sql
-- Use EXISTS (no duplicates)
SELECT c.customer_name
FROM customers c
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id);
```

**Performance:**
- DISTINCT: 200ms (sorts all results)
- EXISTS: 50ms (stops at first match)

---

### OR in WHERE Clause

**Problem:** OR prevents index use in some databases.

**❌ Bad: OR with different columns**

```sql
-- May not use indexes efficiently
SELECT * FROM products 
WHERE category = 'Electronics' 
   OR price < 100;
```

**✅ Good: UNION or separate conditions**

```sql
-- Use UNION for better index usage
SELECT * FROM products WHERE category = 'Electronics'
UNION
SELECT * FROM products WHERE price < 100;
```

**Alternative: Composite strategy**

```sql
-- If both conditions target same column
SELECT * FROM products WHERE category IN ('Electronics', 'Computers');
-- Uses index efficiently
```

---

### Implicit Type Conversions

**Problem:** Database converts types automatically, preventing index use.

**❌ Bad: Type mismatch**

```sql
-- user_id is INT, but comparing with string
SELECT * FROM users WHERE user_id = '123';  -- Implicit conversion
```

**Impact:**
```
Seq Scan on users  (Filter: ((user_id)::text = '123'::text))
-- Function applied to column, index not used!
```

**✅ Good: Match types**

```sql
SELECT * FROM users WHERE user_id = 123;  -- Correct type
-- Index Scan using idx_user_id on users
```

**Common Cases:**

| Column Type | ❌ Bad | ✅ Good |
|-------------|-------|---------|
| INT | `WHERE id = '123'` | `WHERE id = 123` |
| VARCHAR | `WHERE name = 123` | `WHERE name = '123'` |
| DATE | `WHERE date = '2024-01-01'` | `WHERE date = DATE '2024-01-01'` |

---

## When NOT to Use Subqueries

### Correlated Subqueries in SELECT

**Problem:** Executes for every row in outer query.

**❌ Bad: Correlated subquery in SELECT**

```sql
SELECT 
    e.name,
    (SELECT AVG(salary) FROM employees WHERE department_id = e.department_id) AS avg_dept_salary
FROM employees e;
```

**Execution:** Subquery runs once for EACH employee!

**For 1000 employees:**
- 1000 subquery executions
- Very slow

**✅ Good: Use JOIN or Window Function**

```sql
-- Option 1: JOIN
SELECT e.name, d.avg_salary
FROM employees e
JOIN (
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
) d ON e.department_id = d.department_id;

-- Option 2: Window function (better)
SELECT 
    name,
    AVG(salary) OVER (PARTITION BY department_id) AS avg_dept_salary
FROM employees;
```

**Performance:**

| Approach | Executions | Time |
|----------|-----------|------|
| Correlated subquery | 1000 | 2000ms |
| JOIN | 2 | 150ms |
| Window function | 1 | 50ms |

---

### Repeated Subqueries

**❌ Bad: Same subquery multiple times**

```sql
SELECT 
    name,
    salary,
    salary / (SELECT AVG(salary) FROM employees) AS ratio,
    salary - (SELECT AVG(salary) FROM employees) AS diff
FROM employees;
```

**Problem:** Subquery calculated twice for every row!

**✅ Good: Use CTE**

```sql
WITH avg_salary AS (
    SELECT AVG(salary) AS avg_sal FROM employees
)
SELECT 
    name,
    salary,
    salary / avg_sal AS ratio,
    salary - avg_sal AS diff
FROM employees, avg_salary;
```

**Performance:**
- Repeated subquery: 1000ms (calculated 2000 times)
- CTE: 50ms (calculated once)

---

### Subqueries vs Window Functions

**❌ Bad: Subquery for ranking**

```sql
SELECT 
    name,
    salary,
    (SELECT COUNT(*) 
     FROM employees e2 
     WHERE e2.salary > e1.salary) + 1 AS rank
FROM employees e1;
```

**✅ Good: Window function**

```sql
SELECT 
    name,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS rank
FROM employees;
```

**Performance:**

| Approach | Complexity | Time (1000 rows) |
|----------|-----------|------------------|
| Subquery | O(n²) | 5000ms |
| Window function | O(n log n) | 100ms |

---

## JOIN vs EXISTS

### Performance Comparison

**Scenario:** Find customers who have placed orders.

**Using JOIN:**

```sql
SELECT DISTINCT c.customer_id, c.name
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id;
```

**Using EXISTS:**

```sql
SELECT c.customer_id, c.name
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);
```

**Sample Data:**

**customers:**
| customer_id | name |
|-------------|------|
| 1 | Alice |
| 2 | Bob |
| 3 | Carol |

**orders:**
| order_id | customer_id |
|----------|-------------|
| 101 | 1 |
| 102 | 1 |
| 103 | 2 |

**Results:** Both return same data

| customer_id | name |
|-------------|------|
| 1 | Alice |
| 2 | Bob |

---

### When to Use JOIN

**Use JOIN when:**

1. **Need columns from both tables**
   ```sql
   -- Need order details
   SELECT c.name, o.order_date, o.total
   FROM customers c
   JOIN orders o ON c.customer_id = o.customer_id;
   ```

2. **Aggregate data from related table**
   ```sql
   -- Count orders per customer
   SELECT c.name, COUNT(o.order_id) AS order_count
   FROM customers c
   LEFT JOIN orders o ON c.customer_id = o.customer_id
   GROUP BY c.customer_id, c.name;
   ```

3. **Need data from multiple related rows**
   ```sql
   -- All orders with customer names
   SELECT c.name, o.order_id, o.total
   FROM customers c
   JOIN orders o ON c.customer_id = o.customer_id;
   ```

---

### When to Use EXISTS

**Use EXISTS when:**

1. **Only checking for existence** (yes/no)
   ```sql
   -- Customers who have placed ANY order
   SELECT c.name
   FROM customers c
   WHERE EXISTS (SELECT 1 FROM orders WHERE customer_id = c.customer_id);
   ```

2. **Large related table**
   - EXISTS stops at first match (short-circuits)
   - JOIN processes all matching rows

3. **Avoiding DISTINCT**
   ```sql
   -- ❌ JOIN requires DISTINCT
   SELECT DISTINCT c.name
   FROM customers c
   JOIN orders o ON c.customer_id = o.customer_id;
   
   -- ✅ EXISTS no duplicates
   SELECT c.name
   FROM customers c
   WHERE EXISTS (SELECT 1 FROM orders WHERE customer_id = c.customer_id);
   ```

**Performance Comparison:**

| Scenario | JOIN | EXISTS | Winner |
|----------|------|--------|--------|
| Need related columns | Fast | N/A | JOIN |
| Only checking existence | Slow (needs DISTINCT) | Fast (stops early) | EXISTS |
| 1:1 relationship | Similar | Similar | Either |
| 1:Many relationship | Slow (duplicates) | Fast | EXISTS |

**Example - Large Table:**

```sql
-- orders table has 1M rows, customers has 1K rows
-- Each customer has 1000 orders on average

-- JOIN approach: processes 1M order rows
SELECT DISTINCT c.name
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id;
-- Time: 5000ms

-- EXISTS approach: stops at first order for each customer
SELECT c.name
FROM customers c
WHERE EXISTS (SELECT 1 FROM orders WHERE customer_id = c.customer_id LIMIT 1);
-- Time: 50ms (100x faster!)
```

---

## CTE vs Subquery Performance

### Materialization Differences

**CTE (WITH clause):**
```sql
WITH high_earners AS (
    SELECT * FROM employees WHERE salary > 100000
)
SELECT * FROM high_earners WHERE department_id = 10;
```

**Equivalent Subquery:**
```sql
SELECT * FROM (
    SELECT * FROM employees WHERE salary > 100000
) AS high_earners
WHERE department_id = 10;
```

---

### Optimization Behavior

**PostgreSQL/MySQL:**
- **CTEs**: Can be materialized (executed once, results cached)
- **Subqueries**: Often inlined and optimized

**SQL Server:**
- Both CTEs and subqueries optimized similarly

**Example:**

**CTE - May Materialize:**
```sql
WITH dept_avg AS (
    SELECT department_id, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department_id
)
SELECT e.name, d.avg_sal
FROM employees e
JOIN dept_avg d ON e.department_id = d.department_id
WHERE e.salary > d.avg_sal;
```

**Execution:**
1. Execute CTE, store results
2. JOIN with stored results

**Subquery - May Inline:**
```sql
SELECT e.name, d.avg_sal
FROM employees e
JOIN (
    SELECT department_id, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department_id
) d ON e.department_id = d.department_id
WHERE e.salary > d.avg_sal;
```

**Execution:**
1. Optimizer may push WHERE condition into subquery
2. More optimization opportunities

---

### When to Use Each

| Situation | Use CTE | Use Subquery |
|-----------|---------|--------------|
| **Readability** | ✅ Better for complex queries | ❌ Can be nested/messy |
| **Reused multiple times** | ✅ Referenced multiple times | ❌ Executed multiple times |
| **Simple, one-time use** | ❌ Overhead | ✅ Better optimization |
| **Recursive queries** | ✅ Required | ❌ Not supported |
| **Need optimization hints** | ❌ Limited control | ✅ More flexible |

**Example - CTE Better (Reuse):**

```sql
-- ✅ Good: CTE calculated once, used twice
WITH top_products AS (
    SELECT product_id, revenue
    FROM sales
    GROUP BY product_id
    ORDER BY revenue DESC
    LIMIT 10
)
SELECT 'Revenue' AS metric, SUM(revenue) FROM top_products
UNION ALL
SELECT 'Count' AS metric, COUNT(*) FROM top_products;
```

**Example - Subquery Better (Simple):**

```sql
-- ✅ Good: Simple filter, optimizer can inline
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
-- Subquery runs once, result used in comparison
```

**Performance Test Results:**

| Query Type | Execution Time | Use Case |
|------------|----------------|----------|
| CTE (reused 3 times) | 100ms | Multiple references |
| Subquery (repeated 3 times) | 300ms | Multiple references |
| Simple subquery | 50ms | One-time use |
| Simple CTE | 52ms | One-time use |

---

## Best Practices Checklist

### Indexing
- [ ] Index foreign key columns
- [ ] Index columns used in WHERE, JOIN, ORDER BY
- [ ] Use composite indexes with proper column order (most selective first)
- [ ] Avoid indexing low-cardinality columns
- [ ] Remove unused indexes
- [ ] Monitor index usage with database tools

### Query Writing
- [ ] Select only needed columns (avoid SELECT *)
- [ ] Use WHERE before JOIN when possible
- [ ] Avoid functions on indexed columns in WHERE
- [ ] Use EXISTS instead of DISTINCT with JOIN
- [ ] Limit result sets with LIMIT/TOP
- [ ] Use UNION ALL instead of UNION when duplicates OK

### Performance Analysis
- [ ] Use EXPLAIN before optimizing
- [ ] Test with production-like data volumes
- [ ] Monitor slow query logs
- [ ] Set up query performance monitoring
- [ ] Benchmark before and after changes
- [ ] Profile end-to-end query time

### Anti-Pattern Avoidance
- [ ] Avoid N+1 queries (use JOINs)
- [ ] Avoid correlated subqueries in SELECT
- [ ] Avoid OR conditions on different columns
- [ ] Avoid implicit type conversions
- [ ] Avoid SELECT DISTINCT as a band-aid
- [ ] Avoid functions on indexed columns

### Advanced Optimization
- [ ] Use window functions instead of subqueries
- [ ] Use CTEs for complex, reused queries
- [ ] Consider query caching strategies
- [ ] Implement pagination properly (OFFSET can be slow)
- [ ] Use batch operations for bulk updates
- [ ] Consider read replicas for heavy read workloads

---

## Advanced Interview Questions

### Q1: How do indexes improve performance?

**Answer:**

Indexes create a separate data structure (usually B-Tree) that allows quick lookups.

**Without Index:**
- Sequential scan: O(n)
- Read every row to find matches
- 1M rows = 1M reads

**With Index:**
- B-Tree search: O(log n)
- Navigate tree to find matches
- 1M rows = ~20 reads

**Trade-offs:**
- ✅ Faster reads (SELECT)
- ❌ Slower writes (INSERT/UPDATE/DELETE)
- ❌ Additional storage

**Example:**
```sql
-- Without index: 2000ms (scans 1M rows)
SELECT * FROM users WHERE email = 'alice@example.com';

-- With index: 5ms (uses B-Tree)
CREATE INDEX idx_email ON users(email);
SELECT * FROM users WHERE email = 'alice@example.com';
```

---

### Q2: Explain when EXISTS is better than JOIN

**Answer:**

**EXISTS is better when:**

1. **Only checking for existence**
   - EXISTS stops at first match (short-circuits)
   - JOIN processes all matches

2. **1:Many relationship with many matches**
   - JOIN creates duplicates (needs DISTINCT)
   - EXISTS returns each row once

3. **Large related table**
   - EXISTS can use index and stop
   - JOIN may process all related rows

**Example:**

```sql
-- Customer has 1000 orders

-- JOIN: processes 1000 order rows per customer
SELECT DISTINCT c.name
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id;
-- Result: 1000 rows processed, DISTINCT applied

-- EXISTS: stops after first order found
SELECT c.name
FROM customers c
WHERE EXISTS (SELECT 1 FROM orders WHERE customer_id = c.customer_id);
-- Result: 1 row found, stops immediately
```

**Performance:**
- JOIN + DISTINCT: 1000ms
- EXISTS: 10ms (100x faster)

---

### Q3: What's the difference between Seq Scan and Index Scan?

**Answer:**

| Aspect | Seq Scan | Index Scan |
|--------|----------|------------|
| **Method** | Reads table sequentially | Uses index to find rows |
| **Speed** | O(n) | O(log n) + row fetch |
| **When used** | No index, or small table | Index available |
| **Disk I/O** | High | Low (for selective queries) |
| **CPU** | Low | Higher (index traversal) |

**Seq Scan is faster when:**
- Returning >15-20% of rows (index overhead not worth it)
- Small tables (<1000 rows)
- No suitable index

**Example:**

```sql
EXPLAIN SELECT * FROM users WHERE created_at > '2024-01-01';
-- Returns 80% of rows
-- Result: Seq Scan (faster than index for bulk reads)

EXPLAIN SELECT * FROM users WHERE user_id = 123;
-- Returns 1 row
-- Result: Index Scan (much faster)
```

---

### Q4: How does composite index column order matter?

**Answer:**

**Composite index is like a phone book: (Last Name, First Name)**

```sql
CREATE INDEX idx_name ON users(last_name, first_name);
```

**Works like:**
1. Index sorted by last_name first
2. Within same last_name, sorted by first_name

**Query patterns:**

| Query | Index Used? | Why |
|-------|-------------|-----|
| `WHERE last_name = 'Smith'` | ✅ Yes | Uses leftmost column |
| `WHERE last_name = 'Smith' AND first_name = 'John'` | ✅ Yes | Uses both columns |
| `WHERE first_name = 'John'` | ❌ No | Skips leftmost column |

**Analogy:**
- Finding "Smith, John" in phone book: Easy (start with Smith)
- Finding all "John" (any last name): Hard (must scan entire book)

**Optimization tip:**
```sql
-- Put most selective column first
CREATE INDEX idx_selective ON orders(customer_id, order_date);
-- If filtering by customer_id is more selective than order_date
```

---

### Q5: When should you NOT use an index?

**Answer:**

**Don't index when:**

1. **Small tables** (<1000 rows)
   - Full scan is faster

2. **Low cardinality** (few distinct values)
   ```sql
   -- ❌ Bad: Only 2 values (M/F)
   CREATE INDEX idx_gender ON users(gender);
   ```

3. **Frequent updates on indexed columns**
   - Each UPDATE requires index update
   - Slows down writes significantly

4. **Queries return large % of rows** (>20%)
   - Index lookup overhead not worth it
   - Seq scan is faster

5. **Wide columns** (TEXT, BLOB)
   - Index size becomes huge
   - Use full-text search instead

**Example:**

```sql
-- Table: logs (100M rows, heavily inserted)
-- Column: severity (values: INFO, WARN, ERROR)

-- ❌ Bad: Index on low-cardinality column with heavy writes
CREATE INDEX idx_severity ON logs(severity);

-- Each INSERT updates index
-- 90% of queries are INSERTs
-- Index slows down all writes for minimal read benefit
```

---

## Summary

### Performance Optimization Hierarchy

1. **Schema Design** (75% of performance)
   - Proper normalization
   - Appropriate indexes
   - Correct data types

2. **Query Writing** (20% of performance)
   - Efficient queries
   - Avoid anti-patterns
   - Use appropriate techniques

3. **Database Configuration** (5% of performance)
   - Buffer sizes
   - Connection pools
   - Hardware resources

### Quick Reference Table

| Technique | Use Case | Speedup | Complexity |
|-----------|----------|---------|------------|
| **Add index** | Selective WHERE/JOIN | 100-1000x | Low |
| **Use EXISTS** | Check existence | 10-100x | Low |
| **Window function** | Replace subquery | 50x | Medium |
| **Covering index** | Avoid table lookup | 5-10x | Low |
| **CTE for reuse** | Multiple references | 2-3x | Low |
| **Avoid SELECT *** | Reduce data transfer | 2-10x | Low |

### Key Takeaways

1. **Measure first** - Use EXPLAIN before optimizing
2. **Index strategically** - Foreign keys, WHERE, JOIN, ORDER BY
3. **Avoid anti-patterns** - N+1, SELECT *, correlated subqueries
4. **Choose right technique** - JOIN vs EXISTS, CTE vs subquery
5. **Test with real data** - Small datasets hide problems
6. **Balance reads vs writes** - More indexes = slower writes
7. **Profile and monitor** - Continuous performance monitoring

**Master these techniques to write production-grade SQL!** 🚀
