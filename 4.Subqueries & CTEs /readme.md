# Subqueries & CTEs - Complete Guide

## Index
1. [Introduction](#introduction)
2. [Subqueries](#subqueries)
   - [Scalar Subqueries](#scalar-subqueries)
   - [Correlated Subqueries](#correlated-subqueries)
   - [Subqueries in SELECT](#subqueries-in-select)
   - [Subqueries in FROM](#subqueries-in-from)
   - [Subqueries in WHERE](#subqueries-in-where)
3. [CTEs (Common Table Expressions)](#ctes-common-table-expressions)
   - [Simple CTEs](#simple-ctes)
   - [Multiple CTEs](#multiple-ctes)
   - [Recursive CTEs](#recursive-ctes)
4. [Comparison: Subqueries vs CTEs](#comparison-subqueries-vs-ctes)
5. [Best Practices](#best-practices)

---

## Introduction

**Subqueries** and **CTEs (Common Table Expressions)** are powerful SQL features that allow you to break complex queries into manageable parts.

- **Subquery**: A query nested inside another query
- **CTE**: A temporary named result set that exists only during query execution

---

## Subqueries

A subquery is a query within another SQL query. It can return:
- A single value (scalar)
- A single row
- A single column
- A complete table

### Scalar Subqueries

A **scalar subquery** returns exactly **one value** (single row, single column).

**Theory:**
- Used where a single value is expected
- Often used for comparisons or calculations
- Executes once and returns one result

**Example:**

```sql
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

**Sample Data - employees table:**

| id | name | salary | dept_id |
|----|------|--------|---------|
| 1 | Alice | 60000 | 10 |
| 2 | Bob | 75000 | 20 |
| 3 | Carol | 55000 | 10 |

**Output:**

| name | salary |
|------|--------|
| Bob | 75000 |

*Explanation:* The subquery `(SELECT AVG(salary) FROM employees)` returns 63333. Only Bob's salary exceeds this average.

---

### Correlated Subqueries

A **correlated subquery** references columns from the outer query. It executes **once for each row** in the outer query.

**Theory:**
- Depends on the outer query for its values
- Executes repeatedly (once per outer row)
- Uses values from the current row of outer query

**Example:**

```sql
SELECT e1.name, e1.salary
FROM employees e1
WHERE e1.salary > (SELECT AVG(e2.salary) 
                   FROM employees e2 
                   WHERE e2.dept_id = e1.dept_id);
```

**Sample Data - employees table:**

| id | name | salary | dept_id |
|----|------|--------|---------|
| 1 | Alice | 60000 | 10 |
| 2 | Bob | 75000 | 20 |
| 3 | Carol | 55000 | 10 |
| 4 | David | 70000 | 20 |

**Output:**

| name | salary |
|------|--------|
| Alice | 60000 |
| Bob | 75000 |

*Explanation:* Alice's salary (60000) is compared to average of dept 10 (57500). Bob's salary (75000) is compared to average of dept 20 (72500).

---

### Subqueries in SELECT

Subqueries in the **SELECT clause** add calculated columns to the result.

**Theory:**
- Returns a value for each row in the result set
- Often scalar or correlated subqueries
- Creates computed columns

**Example:**

```sql
SELECT name, 
       salary,
       (SELECT AVG(salary) FROM employees) AS avg_salary
FROM employees;
```

**Sample Data - employees table:**

| id | name | salary |
|----|------|--------|
| 1 | Alice | 60000 |
| 2 | Bob | 75000 |

**Output:**

| name | salary | avg_salary |
|------|--------|------------|
| Alice | 60000 | 67500 |
| Bob | 75000 | 67500 |

*Explanation:* The subquery calculates the average salary once and displays it alongside each employee's data.

---

### Subqueries in FROM

Subqueries in the **FROM clause** create temporary tables (also called **derived tables** or **inline views**).

**Theory:**
- Acts as a virtual table
- Must have an alias
- Can simplify complex queries by breaking them into steps

**Example:**

```sql
SELECT dept_id, avg_sal
FROM (SELECT dept_id, AVG(salary) AS avg_sal
      FROM employees
      GROUP BY dept_id) AS dept_avg
WHERE avg_sal > 60000;
```

**Sample Data - employees table:**

| id | name | salary | dept_id |
|----|------|--------|---------|
| 1 | Alice | 60000 | 10 |
| 2 | Bob | 75000 | 20 |
| 3 | Carol | 55000 | 10 |

**Intermediate Result (dept_avg):**

| dept_id | avg_sal |
|---------|---------|
| 10 | 57500 |
| 20 | 75000 |

**Final Output:**

| dept_id | avg_sal |
|---------|---------|
| 20 | 75000 |

*Explanation:* The subquery groups by department and calculates averages. The outer query filters departments with average salary > 60000.

---

### Subqueries in WHERE

Subqueries in the **WHERE clause** filter rows based on results from another query.

**Theory:**
- Used with operators: `IN`, `NOT IN`, `EXISTS`, `ANY`, `ALL`
- Can return single or multiple values
- Most common use of subqueries

**Example with IN:**

```sql
SELECT name, salary
FROM employees
WHERE dept_id IN (SELECT dept_id 
                  FROM departments 
                  WHERE location = 'NYC');
```

**Sample Data:**

**departments table:**
| dept_id | dept_name | location |
|---------|-----------|----------|
| 10 | Sales | NYC |
| 20 | IT | LA |

**employees table:**
| id | name | salary | dept_id |
|----|------|--------|---------|
| 1 | Alice | 60000 | 10 |
| 2 | Bob | 75000 | 20 |

**Output:**

| name | salary |
|------|--------|
| Alice | 60000 |

*Explanation:* The subquery returns dept_id 10 (NYC location). Only Alice works in dept 10.

**Example with EXISTS:**

```sql
SELECT name
FROM employees e
WHERE EXISTS (SELECT 1 
              FROM orders o 
              WHERE o.emp_id = e.id);
```

**Sample Data:**

**orders table:**
| order_id | emp_id |
|----------|--------|
| 101 | 1 |

**Output:**

| name |
|------|
| Alice |

*Explanation:* EXISTS checks if Alice has any orders. Returns TRUE for Alice, FALSE for others.

---

## CTEs (Common Table Expressions)

CTEs are temporary named result sets defined using the `WITH` keyword. They improve readability and can be referenced multiple times.

### Simple CTEs

A **simple CTE** defines one temporary result set.

**Theory:**
- Defined with `WITH cte_name AS (query)`
- Exists only during query execution
- Improves code readability
- Can be referenced like a table

**Example:**

```sql
WITH high_earners AS (
    SELECT name, salary, dept_id
    FROM employees
    WHERE salary > 60000
)
SELECT * FROM high_earners;
```

**Sample Data - employees table:**

| id | name | salary | dept_id |
|----|------|--------|---------|
| 1 | Alice | 60000 | 10 |
| 2 | Bob | 75000 | 20 |
| 3 | Carol | 65000 | 10 |

**Output:**

| name | salary | dept_id |
|------|--------|---------|
| Bob | 75000 | 20 |
| Carol | 65000 | 10 |

*Explanation:* The CTE `high_earners` filters employees with salary > 60000, then the main query selects from it.

---

### Multiple CTEs

You can define **multiple CTEs** in a single query, separated by commas.

**Theory:**
- Define multiple temporary result sets
- Each CTE can reference previously defined CTEs
- Syntax: `WITH cte1 AS (...), cte2 AS (...)`

**Example:**

```sql
WITH 
dept_avg AS (
    SELECT dept_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY dept_id
),
high_avg_depts AS (
    SELECT dept_id
    FROM dept_avg
    WHERE avg_salary > 60000
)
SELECT e.name, e.salary
FROM employees e
JOIN high_avg_depts h ON e.dept_id = h.dept_id;
```

**Sample Data - employees table:**

| id | name | salary | dept_id |
|----|------|--------|---------|
| 1 | Alice | 55000 | 10 |
| 2 | Bob | 75000 | 20 |
| 3 | Carol | 70000 | 20 |

**Intermediate Results:**

**dept_avg CTE:**
| dept_id | avg_salary |
|---------|------------|
| 10 | 55000 |
| 20 | 72500 |

**high_avg_depts CTE:**
| dept_id |
|---------|
| 20 |

**Final Output:**

| name | salary |
|------|--------|
| Bob | 75000 |
| Carol | 70000 |

*Explanation:* First CTE calculates department averages. Second CTE filters departments with avg > 60000. Main query retrieves employees from those departments.

---

### Recursive CTEs

**Recursive CTEs** reference themselves to process hierarchical or iterative data.

**Theory:**
- Contains two parts: **anchor member** (base case) and **recursive member**
- Uses `UNION ALL` to combine parts
- Continues until no new rows are generated
- Common uses: organizational charts, hierarchies, graphs

**Structure:**
```sql
WITH RECURSIVE cte_name AS (
    -- Anchor member (base case)
    SELECT ...
    UNION ALL
    -- Recursive member (references cte_name)
    SELECT ... FROM cte_name ...
)
SELECT * FROM cte_name;
```

**Example - Number sequence:**

```sql
WITH RECURSIVE numbers AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1
    FROM numbers
    WHERE n < 5
)
SELECT * FROM numbers;
```

**Output:**

| n |
|---|
| 1 |
| 2 |
| 3 |
| 4 |
| 5 |

*Explanation:* Starts with 1 (anchor), then recursively adds 1 until reaching 5.

**Example - Employee hierarchy:**

```sql
WITH RECURSIVE emp_hierarchy AS (
    -- Anchor: Start with CEO (no manager)
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive: Get employees who report to current level
    SELECT e.id, e.name, e.manager_id, eh.level + 1
    FROM employees e
    JOIN emp_hierarchy eh ON e.manager_id = eh.id
)
SELECT * FROM emp_hierarchy;
```

**Sample Data - employees table:**

| id | name | manager_id |
|----|------|------------|
| 1 | Alice | NULL |
| 2 | Bob | 1 |
| 3 | Carol | 1 |
| 4 | David | 2 |

**Output:**

| id | name | manager_id | level |
|----|------|------------|-------|
| 1 | Alice | NULL | 1 |
| 2 | Bob | 1 | 2 |
| 3 | Carol | 1 | 2 |
| 4 | David | 2 | 3 |

*Explanation:* Starts with Alice (CEO, level 1). Finds her direct reports (Bob, Carol at level 2). Then finds Bob's reports (David at level 3).

---

## Comparison: Subqueries vs CTEs

| Feature | Subqueries | CTEs |
|---------|-----------|------|
| **Readability** | Can be hard to read when nested | More readable, named components |
| **Reusability** | Cannot be referenced multiple times | Can be referenced multiple times |
| **Recursion** | Not supported | Supported (Recursive CTEs) |
| **Performance** | Generally similar | Generally similar |
| **Best for** | Simple, one-time calculations | Complex queries, hierarchies |

---

## Best Practices

### Subqueries
1. **Use scalar subqueries** when you need a single value
2. **Avoid correlated subqueries** in large tables (can be slow)
3. **Consider EXISTS** instead of IN for better performance with large datasets
4. **Always alias** subqueries in FROM clause

### CTEs
1. **Use CTEs** for complex queries to improve readability
2. **Name CTEs meaningfully** to describe their purpose
3. **Break complex logic** into multiple CTEs
4. **Use recursive CTEs** for hierarchical data
5. **Remember**: CTEs are executed only once, even if referenced multiple times

### General
- **Choose CTEs** when query will be reused or is complex
- **Choose subqueries** for simple, one-off calculations
- **Test performance** with both approaches on large datasets
- **Comment your code** especially for recursive CTEs

---

## Summary

- **Scalar subqueries**: Return single values for comparisons
- **Correlated subqueries**: Execute per row, reference outer query
- **Subquery locations**: SELECT (computed columns), FROM (derived tables), WHERE (filtering)
- **Simple CTEs**: Named temporary result sets
- **Multiple CTEs**: Chain multiple temporary results
- **Recursive CTEs**: Process hierarchical/iterative data

Both subqueries and CTEs are essential tools for writing efficient, maintainable SQL queries. Choose based on complexity and readability needs.
