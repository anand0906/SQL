# SQL Joins - Complete Learning Guide

## Table of Contents
1. [Quick Definitions Reference](#-quick-definitions-reference)
2. [Fundamentals](#1-fundamentals)
   - [What are Joins and Why We Need Them](#what-are-joins-and-why-we-need-them)
   - [Primary Keys and Foreign Keys](#primary-keys-and-foreign-keys)
   - [Table Relationships](#table-relationships)
   - [Basic JOIN Syntax](#basic-join-syntax)
3. [INNER JOIN](#2-inner-join)
4. [LEFT JOIN (LEFT OUTER JOIN)](#3-left-join-left-outer-join)
5. [RIGHT JOIN (RIGHT OUTER JOIN)](#4-right-join-right-outer-join)
6. [FULL OUTER JOIN](#5-full-outer-join)
7. [CROSS JOIN](#6-cross-join)
8. [SELF JOIN](#7-self-join)
9. [NATURAL JOIN](#8-natural-join)
10. [Visual Summary](#9-visual-summary)

---

## 📖 Quick Definitions Reference

Before diving into details, here's a comprehensive overview of all JOIN types:

## 📖 Quick Definitions Reference

### 🔹 INNER JOIN
**Retrieves only the records that have matching values in both tables based on the specified join condition**
- It only matches the columns mentioned in the JOIN condition, not every column from both tables
- Records that do not have a match in both tables are excluded from the result
- **Formula:** `A ∩ B` (Intersection)

### 🔹 LEFT JOIN (LEFT OUTER JOIN)
**First performs an INNER JOIN to fetch all matching records from both tables**
- Additionally, it includes any remaining records from the left table that did not find a match in the right table
- For columns from the right table that do not have a match, NULL values will be returned
- The left table is the one mentioned before the LEFT JOIN keyword
- **Formula:** `INNER JOIN + Unmatched Left Records`

### 🔹 RIGHT JOIN (RIGHT OUTER JOIN)
**Similar to LEFT JOIN, it first performs an INNER JOIN to get all matching records**
- It then includes any remaining records from the right table that were not part of the initial inner join
- The right table is the one mentioned after the RIGHT JOIN keyword, and it is given priority
- **Formula:** `INNER JOIN + Unmatched Right Records`

### 🔹 FULL OUTER JOIN
**Combines the results of both LEFT JOIN and RIGHT JOIN**
- Returns all matching records between two tables
- Plus all unmatched records from the left table
- Plus all unmatched records from the right table
- Brings together all data where there's a match, and includes any remaining data from both tables where there isn't a match
- **Formula:** `LEFT JOIN + RIGHT JOIN`

### 🔹 CROSS JOIN
**Returns the Cartesian product of the two tables**
- Also known as Cartesian Join
- Every row from the first table is combined with every row from the second table
- Does not require a join condition
- **Formula:** `A × B` = M rows × N rows = M×N total rows

### 🔹 SELF JOIN
**A regular JOIN (like INNER JOIN, LEFT JOIN, or RIGHT JOIN) where a table is joined with itself**
- Used when you need to match records within the same table
- Typically to find relationships between data points in a single table (parent-child, manager-employee relationships)
- Requires aliases for the table to distinguish between the two instances
- **Formula:** `Table A JOIN Table A` (using aliases)

### 🔹 NATURAL JOIN
**Automatically joins two tables based on columns that have the same name in both tables**
- If there are no common columns, it behaves like a CROSS JOIN
- ⚠️ **NOT RECOMMENDED**: Gives control to SQL to decide join columns, which can lead to unexpected or incorrect results
- **Formula:** `Auto JOIN on matching column names`

### 🎯 How JOINs Build on Each Other

```
INNER JOIN (Base)
    ↓
    ├─→ LEFT JOIN = INNER JOIN + Unmatched Left Records
    ├─→ RIGHT JOIN = INNER JOIN + Unmatched Right Records
    └─→ FULL OUTER = LEFT JOIN + RIGHT JOIN = All Records

CROSS JOIN (Independent) = All Combinations (No condition)

SELF JOIN (Technique) = Any JOIN type applied to same table

NATURAL JOIN (Automatic) = Auto INNER JOIN on matching column names
```

---

## 1. Fundamentals

### What are Joins and Why We Need Them

**Joins** are SQL operations that combine rows from two or more tables based on a related column between them. They are essential because:

- **Data Normalization**: To avoid redundancy, we split data into multiple related tables
- **Efficient Storage**: Storing data once and referencing it saves space
- **Data Integrity**: Updates in one place reflect everywhere
- **Complex Queries**: Retrieve related information from multiple sources

**Example Scenario**: Instead of storing customer details with every order, we store customers once and reference them in orders.

---

### Primary Keys and Foreign Keys

**Primary Key (PK)**:
- Uniquely identifies each record in a table
- Cannot contain NULL values
- Each table has only ONE primary key

**Foreign Key (FK)**:
- A field in one table that references the Primary Key in another table
- Creates relationships between tables
- Can contain NULL values (unless specified otherwise)

**Example Tables**:

```sql
-- Customers table
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(100)
);

-- Orders table
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,  -- Foreign Key
    order_date DATE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

---

### Table Relationships

#### **One-to-One (1:1)**
One record in Table A relates to exactly one record in Table B.

**Example**: Person → Passport

```
Person Table          Passport Table
+------------+        +-------------+
| person_id  |   →    | passport_id |
| name       |        | person_id   |
+------------+        | number      |
                      +-------------+
```

#### **One-to-Many (1:M)** - Most Common
One record in Table A can relate to multiple records in Table B.

**Example**: Customer → Orders

```
Customer Table        Orders Table
+-------------+       +-----------+
| customer_id |   →   | order_id  |
| name        |       | customer_id (FK)
+-------------+       | date      |
                      +-----------+
```

#### **Many-to-Many (M:N)**
Multiple records in Table A relate to multiple records in Table B. Requires a junction/bridge table.

**Example**: Students ↔ Courses (via Enrollments)

```
Students         Enrollments        Courses
+------------+   +-------------+   +-----------+
| student_id |←→ | student_id  |←→ | course_id |
| name       |   | course_id   |   | title     |
+------------+   +-------------+   +-----------+
```

---

### Basic JOIN Syntax

```sql
SELECT columns
FROM table1
JOIN_TYPE table2
ON table1.column = table2.column;
```

**Components**:
- `SELECT`: Columns to retrieve
- `FROM`: First (left) table
- `JOIN_TYPE`: Type of join (INNER, LEFT, RIGHT, etc.)
- `ON`: Condition that defines how tables relate

---

## 2. INNER JOIN

### 📘 Definition

> **This join retrieves only the records that have matching values in both tables based on the specified join condition.**
> 
> **Key Points:**
> - It only matches the columns mentioned in the `JOIN` condition, not every column from both tables
> - Records that do not have a match in both tables are excluded from the result

### 🔢 Formula

```
INNER JOIN = A ∩ B (Intersection)
Result = Only records where A.key = B.key
```

```
Table A          Table B          Result
+-----+          +-----+          +-----+
|  A  |          |  B  |          | A∩B |
|  ●  |    ∩     |  ●  |    =     |  ●  |
+-----+          +-----+          +-----+
```

### Basic Syntax

```sql
SELECT columns
FROM table1
INNER JOIN table2
ON table1.common_column = table2.common_column;
```

### Example: Joining Two Tables

**Sample Data**:

```sql
-- customers table
| customer_id | name          | city      |
|-------------|---------------|-----------|
| 1           | Alice Johnson | New York  |
| 2           | Bob Smith     | London    |
| 3           | Carol White   | Paris     |
| 4           | David Brown   | Tokyo     |

-- orders table
| order_id | customer_id | product       | amount |
|----------|-------------|---------------|--------|
| 101      | 1           | Laptop        | 1200   |
| 102      | 1           | Mouse         | 25     |
| 103      | 2           | Keyboard      | 75     |
| 104      | 5           | Monitor       | 300    |
```

**Query**:
```sql
SELECT customers.name, orders.product, orders.amount
FROM customers
INNER JOIN orders
ON customers.customer_id = orders.customer_id;
```

**Result**:
```
| name          | product   | amount |
|---------------|-----------|--------|
| Alice Johnson | Laptop    | 1200   |
| Alice Johnson | Mouse     | 25     |
| Bob Smith     | Keyboard  | 75     |
```

**Note**: Carol White and David Brown don't appear (no orders). Order 104 doesn't appear (customer_id 5 doesn't exist).

---

### Using Table Aliases

Aliases make queries shorter and more readable.

```sql
SELECT c.name, o.product, o.amount
FROM customers AS c
INNER JOIN orders AS o
ON c.customer_id = o.customer_id;
```

Or without `AS`:
```sql
SELECT c.name, o.product, o.amount
FROM customers c
INNER JOIN orders o
ON c.customer_id = o.customer_id;
```

---

### Joining Multiple Tables (3+ Tables)

**Sample Data - Adding Products Table**:

```sql
-- products table
| product_id | product_name | category    |
|------------|--------------|-------------|
| 1          | Laptop       | Electronics |
| 2          | Mouse        | Accessories |
| 3          | Keyboard     | Accessories |

-- Updated orders table
| order_id | customer_id | product_id | quantity |
|----------|-------------|------------|----------|
| 101      | 1           | 1          | 1        |
| 102      | 1           | 2          | 2        |
| 103      | 2           | 3          | 1        |
```

**Query - Joining 3 Tables**:
```sql
SELECT c.name, p.product_name, p.category, o.quantity
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
INNER JOIN products p ON o.product_id = p.product_id;
```

**Result**:
```
| name          | product_name | category    | quantity |
|---------------|--------------|-------------|----------|
| Alice Johnson | Laptop       | Electronics | 1        |
| Alice Johnson | Mouse        | Accessories | 2        |
| Bob Smith     | Keyboard     | Accessories | 1        |
```

---

### JOIN with WHERE Clause

Filter results after joining.

```sql
SELECT c.name, o.product_id, o.quantity
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
WHERE o.quantity > 1;
```

**Result**:
```
| name          | product_id | quantity |
|---------------|------------|----------|
| Alice Johnson | 2          | 2        |
```

---

### Self JOIN with INNER JOIN

Joining a table to itself. Useful for hierarchical data or comparing rows within the same table.

**Sample Data - Employees**:

```sql
| emp_id | name          | manager_id |
|--------|---------------|------------|
| 1      | Alice (CEO)   | NULL       |
| 2      | Bob           | 1          |
| 3      | Carol         | 1          |
| 4      | David         | 2          |
| 5      | Eve           | 2          |
```

**Query - Find Each Employee with Their Manager**:
```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e
INNER JOIN employees m
ON e.manager_id = m.emp_id;
```

**Result**:
```
| employee | manager       |
|----------|---------------|
| Bob      | Alice (CEO)   |
| Carol    | Alice (CEO)   |
| David    | Bob           |
| Eve      | Bob           |
```

**Note**: Alice doesn't appear because she has no manager (NULL).

---

## 3. LEFT JOIN (LEFT OUTER JOIN)

### 📘 Definition

> **This join first performs an INNER JOIN to fetch all matching records from both tables.**
>
> **Key Points:**
> - Additionally, it includes any remaining records from the left table that did not find a match in the right table
> - For columns from the right table that do not have a match, `NULL` values will be returned
> - The left table is the one mentioned before the `LEFT JOIN` keyword

### 🔢 Formula

```
LEFT JOIN = INNER JOIN + Unmatched Left Records

Step 1: Perform INNER JOIN (get matches)
Step 2: Add remaining left table records (with NULL for right columns)
```

```
Table A          Table B          Result
+-----+          +-----+          +-------+
|  A  |          |  B  |          | A + ∩ |
|  ●  |    +     |  ●  |    =     |  ●●   |
+-----+          +-----+          +-------+
```

### Basic Syntax

```sql
SELECT columns
FROM table1
LEFT JOIN table2
ON table1.common_column = table2.common_column;
```

### Example with Sample Data

**Using the same customers and orders tables**:

```sql
SELECT c.name, o.product, o.amount
FROM customers c
LEFT JOIN orders o
ON c.customer_id = o.customer_id;
```

**Result**:
```
| name          | product   | amount |
|---------------|-----------|--------|
| Alice Johnson | Laptop    | 1200   |
| Alice Johnson | Mouse     | 25     |
| Bob Smith     | Keyboard  | 75     |
| Carol White   | NULL      | NULL   |
| David Brown   | NULL      | NULL   |
```

**Key Difference**: Carol and David appear with NULL values (no orders).

---

### Understanding NULL Values

NULL indicates no matching row in the right table.

**Use Case - Find Customers with No Orders**:
```sql
SELECT c.name
FROM customers c
LEFT JOIN orders o
ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

**Result**:
```
| name         |
|--------------|
| Carol White  |
| David Brown  |
```

---

### LEFT JOIN vs INNER JOIN

**Sample Data**:
```
Customers: 4 records
Orders: 3 records (only for customer_id 1 and 2)
```

| JOIN Type   | Rows Returned | Includes Unmatched Left |
|-------------|---------------|-------------------------|
| INNER JOIN  | 3 rows        | No                      |
| LEFT JOIN   | 5 rows        | Yes (with NULL)         |

---

### Multiple LEFT JOINs

**Sample Data - Adding Shipping Table**:

```sql
-- shipping table
| order_id | status    | shipped_date |
|----------|-----------|--------------|
| 101      | Delivered | 2024-01-15   |
| 103      | In Transit| 2024-01-20   |
```

**Query**:
```sql
SELECT c.name, o.product, s.status
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
LEFT JOIN shipping s ON o.order_id = s.order_id;
```

**Result**:
```
| name          | product   | status     |
|---------------|-----------|------------|
| Alice Johnson | Laptop    | Delivered  |
| Alice Johnson | Mouse     | NULL       |
| Bob Smith     | Keyboard  | In Transit |
| Carol White   | NULL      | NULL       |
| David Brown   | NULL      | NULL       |
```

---

## 4. RIGHT JOIN (RIGHT OUTER JOIN)

### 📘 Definition

> **Similar to the LEFT JOIN, it first performs an INNER JOIN to get all matching records.**
>
> **Key Points:**
> - It then includes any remaining records from the right table that were not part of the initial inner join
> - The right table is the one mentioned after the `RIGHT JOIN` keyword, and it is given priority

### 🔢 Formula

```
RIGHT JOIN = INNER JOIN + Unmatched Right Records

Step 1: Perform INNER JOIN (get matches)
Step 2: Add remaining right table records (with NULL for left columns)
```

```
Table A          Table B          Result
+-----+          +-----+          +-------+
|  A  |          |  B  |          | ∩ + B |
|  ●  |    +     |  ●  |    =     |  ●●   |
+-----+          +-----+          +-------+
```

### Basic Syntax

```sql
SELECT columns
FROM table1
RIGHT JOIN table2
ON table1.common_column = table2.common_column;
```

### Example

**Sample Data**:

```sql
-- departments table
| dept_id | dept_name   |
|---------|-------------|
| 1       | Sales       |
| 2       | Marketing   |
| 3       | IT          |

-- employees table
| emp_id | name  | dept_id |
|--------|-------|---------|
| 1      | Alice | 1       |
| 2      | Bob   | 1       |
| 3      | Carol | 2       |
| 4      | David | NULL    |
```

**Query**:
```sql
SELECT e.name, d.dept_name
FROM employees e
RIGHT JOIN departments d
ON e.dept_id = d.dept_id;
```

**Result**:
```
| name  | dept_name   |
|-------|-------------|
| Alice | Sales       |
| Bob   | Sales       |
| Carol | Marketing   |
| NULL  | IT          |
```

**Note**: IT department appears with NULL (no employees).

---

### RIGHT JOIN vs LEFT JOIN

RIGHT JOIN is the mirror image of LEFT JOIN. They can be converted to each other by swapping table order.

**These queries are equivalent**:

```sql
-- RIGHT JOIN
SELECT e.name, d.dept_name
FROM employees e
RIGHT JOIN departments d
ON e.dept_id = d.dept_id;

-- Converted to LEFT JOIN
SELECT e.name, d.dept_name
FROM departments d
LEFT JOIN employees e
ON d.dept_id = e.dept_id;
```

---

### When to Use RIGHT JOIN

**Recommendation**: Use LEFT JOIN instead. Most developers prefer LEFT JOIN because:
- More intuitive (start with main table on left)
- Easier to read and maintain
- RIGHT JOIN can always be rewritten as LEFT JOIN

**Use RIGHT JOIN when**:
- Company coding standards require it
- Maintaining existing code that uses it
- Personal preference (rare)

---

## 5. FULL OUTER JOIN

### 📘 Definition

> **This join combines the results of both LEFT JOIN and RIGHT JOIN.**
>
> **Key Points:**
> - It returns all matching records between two tables
> - Plus all unmatched records from the left table
> - Plus all unmatched records from the right table
> - Essentially, it brings together all data where there's a match, and includes any remaining data from both tables where there isn't a match

### 🔢 Formula

```
FULL OUTER JOIN = LEFT JOIN + RIGHT JOIN

Result = All matching records + Unmatched left + Unmatched right
       = Everything from both tables
```

```
Table A          Table B          Result
+-----+          +-----+          +-------+
|  A  |          |  B  |          |A + B+∩|
|  ●  |    +     |  ●  |    =     | ●●●   |
+-----+          +-----+          +-------+
```

### Basic Syntax

```sql
SELECT columns
FROM table1
FULL OUTER JOIN table2
ON table1.common_column = table2.common_column;
```

### Example

**Sample Data**:

```sql
-- students table
| student_id | name  |
|------------|-------|
| 1          | Alice |
| 2          | Bob   |
| 3          | Carol |

-- enrollments table
| enrollment_id | student_id | course |
|---------------|------------|--------|
| 101           | 1          | Math   |
| 102           | 2          | Science|
| 103           | 4          | History|
```

**Query**:
```sql
SELECT s.name, e.course
FROM students s
FULL OUTER JOIN enrollments e
ON s.student_id = e.student_id;
```

**Result**:
```
| name  | course  |
|-------|---------|
| Alice | Math    |
| Bob   | Science |
| Carol | NULL    |
| NULL  | History |
```

**Interpretation**:
- Alice & Bob: Enrolled students (matched)
- Carol: Student with no enrollment (left unmatched)
- History: Enrollment with no student record (right unmatched)

---

### Finding All Unmatched Records

**Use Case - Find All Discrepancies**:

```sql
SELECT s.name AS unmatched_student, e.course AS unmatched_course
FROM students s
FULL OUTER JOIN enrollments e
ON s.student_id = e.student_id
WHERE s.student_id IS NULL OR e.enrollment_id IS NULL;
```

**Result**:
```
| unmatched_student | unmatched_course |
|-------------------|------------------|
| Carol             | NULL             |
| NULL              | History          |
```

---

### Simulating FULL OUTER JOIN

**Note**: MySQL doesn't support FULL OUTER JOIN directly. You can simulate it using UNION.

```sql
-- Simulated FULL OUTER JOIN for MySQL
SELECT s.name, e.course
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id

UNION

SELECT s.name, e.course
FROM students s
RIGHT JOIN enrollments e ON s.student_id = e.student_id;
```

---

### FULL OUTER JOIN Use Cases

1. **Data Reconciliation**: Finding discrepancies between two datasets
2. **Audit Reports**: Identifying missing or extra records
3. **Data Migration**: Comparing old and new systems
4. **Inventory Management**: Matching products with stock records

---

## 6. CROSS JOIN

### 📘 Definition

> **Also known as a Cartesian Join, this type of join returns the Cartesian product of the two tables.**
>
> **Key Points:**
> - This means every row from the first table is combined with every row from the second table
> - It does not require a join condition

### 🔢 Formula

```
CROSS JOIN = A × B (Cartesian Product)

Result = Every row from Table A × Every row from Table B
       = M rows × N rows = M × N total rows
```

```
Table A (2 rows)   Table B (3 rows)   Result (2×3=6 rows)
+-----+            +-----+            +-------+
|  A  |            |  B  |            | A × B |
| 1,2 |     ×      |a,b,c|     =      | 1a,1b,|
+-----+            +-----+            | 1c,2a,|
                                      | 2b,2c |
                                      +-------+
```

### Basic Syntax

```sql
-- Method 1
SELECT columns
FROM table1
CROSS JOIN table2;

-- Method 2 (same result)
SELECT columns
FROM table1, table2;
```

### Example

**Sample Data**:

```sql
-- sizes table
| size_id | size_name |
|---------|-----------|
| 1       | Small     |
| 2       | Medium    |
| 3       | Large     |

-- colors table
| color_id | color_name |
|----------|------------|
| 1        | Red        |
| 2        | Blue       |
```

**Query**:
```sql
SELECT s.size_name, c.color_name
FROM sizes s
CROSS JOIN colors c;
```

**Result** (3 × 2 = 6 rows):
```
| size_name | color_name |
|-----------|------------|
| Small     | Red        |
| Small     | Blue       |
| Medium    | Red        |
| Medium    | Blue       |
| Large     | Red        |
| Large     | Blue       |
```

---

### Practical Use Cases

#### 1. **Generating Product Variations**
```sql
SELECT CONCAT(s.size_name, ' - ', c.color_name) AS product_variant
FROM sizes s
CROSS JOIN colors c;
```

#### 2. **Creating Test Data**
```sql
-- Generate all possible time slots for a week
SELECT d.day_name, t.time_slot
FROM days d
CROSS JOIN time_slots t;
```

#### 3. **Reporting All Combinations**
```sql
-- All possible employee-project assignments for planning
SELECT e.name, p.project_name
FROM employees e
CROSS JOIN projects p;
```

---

### CROSS JOIN vs Missing JOIN Condition

**Warning**: Forgetting the ON clause in INNER JOIN creates an accidental CROSS JOIN!

```sql
-- Accidental CROSS JOIN (missing ON clause)
SELECT *
FROM customers
INNER JOIN orders;  -- WRONG! Creates Cartesian product

-- Correct INNER JOIN
SELECT *
FROM customers
INNER JOIN orders ON customers.customer_id = orders.customer_id;
```

---

## 7. SELF JOIN

### 📘 Definition

> **This is a regular JOIN (like INNER JOIN, LEFT JOIN, or RIGHT JOIN) where a table is joined with itself.**
>
> **Key Points:**
> - It's used when you need to match records within the same table
> - Typically to find relationships between data points in a single table, such as identifying parent-child relationships within a family tree or manager-employee relationships
> - It requires aliases for the table to distinguish between the two instances of the table

### 🔢 Formula

```
SELF JOIN = Table A JOIN Table A (using different aliases)

Example: employees e JOIN employees m
         WHERE e.manager_id = m.employee_id
```

### Concept and Use Cases

**Common Scenarios**:
1. **Hierarchical Data**: Employee-Manager relationships
2. **Comparing Rows**: Finding related records within the same table
3. **Finding Duplicates**: Matching similar entries
4. **Referential Relationships**: When a table references itself

---

### Example 1: Hierarchical Data (Employee-Manager)

**Sample Data**:

```sql
-- employees table
| emp_id | name       | title          | manager_id |
|--------|------------|----------------|------------|
| 1      | Sarah      | CEO            | NULL       |
| 2      | John       | VP Sales       | 1          |
| 3      | Emma       | VP Marketing   | 1          |
| 4      | Michael    | Sales Rep      | 2          |
| 5      | Lisa       | Marketing Asst | 3          |
```

**Query - List Employees with Their Managers**:
```sql
SELECT 
    e.name AS employee,
    e.title AS employee_title,
    m.name AS manager,
    m.title AS manager_title
FROM employees e
INNER JOIN employees m
ON e.manager_id = m.emp_id;
```

**Result**:
```
| employee | employee_title  | manager | manager_title |
|----------|-----------------|---------|---------------|
| John     | VP Sales        | Sarah   | CEO           |
| Emma     | VP Marketing    | Sarah   | CEO           |
| Michael  | Sales Rep       | John    | VP Sales      |
| Lisa     | Marketing Asst  | Emma    | VP Marketing  |
```

**Note**: Sarah (CEO) doesn't appear because she has no manager.

---

### Example 2: Comparing Rows (Finding Colleagues)

**Query - Find Employees Who Share the Same Manager**:
```sql
SELECT 
    e1.name AS employee1,
    e2.name AS employee2,
    m.name AS shared_manager
FROM employees e1
INNER JOIN employees e2 
    ON e1.manager_id = e2.manager_id
    AND e1.emp_id < e2.emp_id  -- Avoid duplicates
INNER JOIN employees m
    ON e1.manager_id = m.emp_id;
```

**Result**:
```
| employee1 | employee2 | shared_manager |
|-----------|-----------|----------------|
| John      | Emma      | Sarah          |
```

---

### Example 3: Finding Duplicates

**Sample Data - Products**:

```sql
| product_id | name     | category    | price |
|------------|----------|-------------|-------|
| 1          | Laptop   | Electronics | 1000  |
| 2          | Laptop   | Electronics | 1000  |
| 3          | Mouse    | Accessories | 20    |
| 4          | Keyboard | Accessories | 50    |
```

**Query - Find Duplicate Products**:
```sql
SELECT DISTINCT
    p1.product_id AS id1,
    p2.product_id AS id2,
    p1.name,
    p1.price
FROM products p1
INNER JOIN products p2
    ON p1.name = p2.name
    AND p1.price = p2.price
    AND p1.product_id < p2.product_id;
```

**Result**:
```
| id1 | id2 | name   | price |
|-----|-----|--------|-------|
| 1   | 2   | Laptop | 1000  |
```

---

### Self JOIN with Different JOIN Types

**Using LEFT JOIN - Include Employees Without Managers**:

```sql
SELECT 
    e.name AS employee,
    COALESCE(m.name, 'No Manager') AS manager
FROM employees e
LEFT JOIN employees m
ON e.manager_id = m.emp_id;
```

**Result**:
```
| employee | manager      |
|----------|--------------|
| Sarah    | No Manager   |
| John     | Sarah        |
| Emma     | Sarah        |
| Michael  | John         |
| Lisa     | Emma         |
```

---

## 8. NATURAL JOIN

### 📘 Definition

> **This join works by automatically joining two tables based on columns that have the same name in both tables.**
>
> **Key Points:**
> - If there are no common columns, it behaves like a `CROSS JOIN`
> - ⚠️ **NOT RECOMMENDED**: Using NATURAL JOIN gives control to SQL to decide the join columns, which can lead to unexpected or incorrect results if column names are changed or are unintentionally the same but hold different values

### 🔢 Formula

```
NATURAL JOIN = Automatic JOIN on all matching column names

If common columns exist:
  Result = INNER JOIN on all columns with same names

If no common columns:
  Result = CROSS JOIN
```

### Basic Syntax

```sql
SELECT columns
FROM table1
NATURAL JOIN table2;
```

### Example

**Sample Data**:

```sql
-- customers table
| customer_id | name  | city     |
|-------------|-------|----------|
| 1           | Alice | New York |
| 2           | Bob   | London   |

-- orders table
| order_id | customer_id | product |
|----------|-------------|---------|
| 101      | 1           | Laptop  |
| 102      | 2           | Mouse   |
```

**Query**:
```sql
SELECT *
FROM customers
NATURAL JOIN orders;
```

**What Happens**:
- SQL automatically finds `customer_id` (common column)
- Joins on `customer_id` without explicit ON clause

**Result**:
```
| customer_id | name  | city     | order_id | product |
|-------------|-------|----------|----------|---------|
| 1           | Alice | New York | 101      | Laptop  |
| 2           | Bob   | London   | 102      | Mouse   |
```

---

### Why NATURAL JOIN is Risky

**Problem 1: Unexpected Column Matches**

If tables have multiple columns with same names, ALL will be used for joining.

```sql
-- Both tables have 'customer_id' AND 'city'
-- NATURAL JOIN will match on BOTH columns!
-- This might not be what you want.
```

**Problem 2: Schema Changes Break Queries**

If someone adds a column with the same name to one table, your query behavior changes unexpectedly.

**Recommendation**: 
✅ **Use explicit JOIN with ON clause**  
❌ **Avoid NATURAL JOIN in production code**

---

## 9. Visual Summary

### 🔢 Complete Formula Reference

```
┌────────────────┬─────────────────────┬──────────────────────────────────┐
│  JOIN TYPE     │     FORMULA         │         EXPLANATION              │
├────────────────┼─────────────────────┼──────────────────────────────────┤
│ INNER JOIN     │  A ∩ B              │  Intersection (only matches)     │
│                │                     │  = {rows where A.key = B.key}    │
├────────────────┼─────────────────────┼──────────────────────────────────┤
│ LEFT JOIN      │  A + (A ∩ B)        │  All of A + matching B           │
│                │                     │  = A with NULLs for unmatched B  │
├────────────────┼─────────────────────┼──────────────────────────────────┤
│ RIGHT JOIN     │  (A ∩ B) + B        │  All of B + matching A           │
│                │                     │  = B with NULLs for unmatched A  │
├────────────────┼─────────────────────┼──────────────────────────────────┤
│ FULL OUTER     │  A ∪ B              │  Union (everything)              │
│                │  = LEFT ∪ RIGHT     │  = All records from both tables  │
├────────────────┼─────────────────────┼──────────────────────────────────┤
│ CROSS JOIN     │  A × B              │  Cartesian product               │
│                │  = M × N rows       │  = Every combination             │
├────────────────┼─────────────────────┼──────────────────────────────────┤
│ SELF JOIN      │  A ⋈ A              │  Table with itself (aliases)     │
│                │  (A₁ JOIN A₂)       │  = A as e JOIN A as m            │
├────────────────┼─────────────────────┼──────────────────────────────────┤
│ NATURAL JOIN   │  A ⋈ B (auto)       │  Auto match on column names      │
│                │                     │  ⚠️ AVOID IN PRODUCTION          │
└────────────────┴─────────────────────┴──────────────────────────────────┘
```

### Mathematical Relationships

```
INNER JOIN:         A ∩ B
LEFT JOIN:          A ∪ (A ∩ B) = A + matching B
RIGHT JOIN:         (A ∩ B) ∪ B = matching A + B
FULL OUTER JOIN:    A ∪ B = LEFT ∪ RIGHT
CROSS JOIN:         A × B (Cartesian Product)

Special case:
FULL OUTER = INNER + (A - B) + (B - A)
           = Matches + Left Only + Right Only
```

### Row Count Formulas

```
Given:
  Table A has M rows
  Table B has N rows
  Matching rows = K

┌────────────────┬─────────────────────────────────┐
│  JOIN TYPE     │     RESULT ROW COUNT            │
├────────────────┼─────────────────────────────────┤
│ INNER JOIN     │  K (only matching)              │
│ LEFT JOIN      │  M (at least, can be more)      │
│ RIGHT JOIN     │  N (at least, can be more)      │
│ FULL OUTER     │  M + N - K                      │
│ CROSS JOIN     │  M × N                          │
│ SELF JOIN      │  Depends on JOIN type used      │
└────────────────┴─────────────────────────────────┘
```

### JOIN Types Quick Reference

```
Given: Table A (customers) and Table B (orders)

┌─────────────────┬──────────────────────────────────────┐
│   JOIN TYPE     │           RESULT                     │
├─────────────────┼──────────────────────────────────────┤
│ INNER JOIN      │ Only matching rows from both tables  │
│                 │        ●●● (intersection)            │
├─────────────────┼──────────────────────────────────────┤
│ LEFT JOIN       │ All from A + matching from B         │
│                 │        ●●●●● (A complete)            │
├─────────────────┼──────────────────────────────────────┤
│ RIGHT JOIN      │ All from B + matching from A         │
│                 │        ●●●●● (B complete)            │
├─────────────────┼──────────────────────────────────────┤
│ FULL OUTER JOIN │ All rows from both tables            │
│                 │        ●●●●●●● (union)               │
├─────────────────┼──────────────────────────────────────┤
│ CROSS JOIN      │ Every row from A × every row from B  │
│                 │        ●●●●●●●●●●● (Cartesian)       │
├─────────────────┼──────────────────────────────────────┤
│ SELF JOIN       │ Table joined with itself             │
│                 │        ●●● (hierarchical/compare)    │
└─────────────────┴──────────────────────────────────────┘
```

---

### When to Use Each JOIN

| JOIN Type       | Use When                                          | Example Use Case                    |
|-----------------|---------------------------------------------------|-------------------------------------|
| **INNER JOIN**  | You only want matching records                    | Orders with valid customers         |
| **LEFT JOIN**   | You want all records from left + matches          | All customers (with/without orders) |
| **RIGHT JOIN**  | You want all records from right + matches         | (Convert to LEFT JOIN instead)      |
| **FULL OUTER**  | You want all records from both tables             | Data reconciliation/audit           |
| **CROSS JOIN**  | You need all possible combinations                | Product variants (size × color)     |
| **SELF JOIN**   | Comparing rows within same table                  | Employee-manager hierarchy          |
| **NATURAL**     | (Avoid in production)                             | Quick ad-hoc queries only           |

---

### Common Patterns & Best Practices

#### ✅ **DO**
- Use table aliases for readability (`FROM customers c`)
- Specify columns explicitly (`SELECT c.name, o.order_id`)
- Use INNER JOIN when you only need matching records
- Use LEFT JOIN when you need all records from the main table
- Filter with WHERE after the JOIN completes
- Test joins with small datasets first

#### ❌ **DON'T**
- Forget the ON clause (creates accidental CROSS JOIN)
- Use SELECT * in production code
- Use NATURAL JOIN in production
- Mix implicit and explicit join syntax
- Create unnecessary Cartesian products

---

### Performance Tips

1. **Index Foreign Keys**: Dramatically speeds up joins
   ```sql
   CREATE INDEX idx_customer_id ON orders(customer_id);
   ```

2. **Filter Early**: Use WHERE on individual tables before joining when possible

3. **Select Only Needed Columns**: Reduces data transfer
   ```sql
   -- Good
   SELECT c.name, o.order_id
   
   -- Avoid
   SELECT *
   ```

4. **Understand Cardinality**: Know your data volumes
   - 1:1 joins: Same number of rows
   - 1:M joins: Result has M rows
   - M:N joins: Result has M × N rows (use carefully!)

---

### Practice Exercise Data

You can use this sample data to practice:

```sql
-- Create practice tables
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(50),
    city VARCHAR(50)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    product VARCHAR(50),
    amount DECIMAL(10,2),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

-- Insert sample data
INSERT INTO customers VALUES
(1, 'Alice Johnson', 'New York'),
(2, 'Bob Smith', 'London'),
(3, 'Carol White', 'Paris'),
(4, 'David Brown', 'Tokyo');

INSERT INTO orders VALUES
(101, 1, 'Laptop', 1200.00),
(102, 1, 'Mouse', 25.00),
(103, 2, 'Keyboard', 75.00),
(104, 2, 'Monitor', 300.00);
```

---

### Conclusion

Mastering SQL joins is essential for working with relational databases. Start with INNER JOIN and LEFT JOIN (the most common), then expand to other types as needed. Always write explicit, clear join conditions and test with sample data.

**Next Steps**:
1. Practice with the sample data above
2. Experiment with different JOIN types
3. Try combining multiple joins
4. Work on real-world scenarios in your projects

Happy querying! 🚀
