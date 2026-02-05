# SQL Fundamentals - Complete Guide

## Table of Contents
1. [What is SQL?](#1-what-is-sql)
2. [RDBMS Concepts](#2-rdbms-concepts)
3. [Tables, Rows, and Columns](#3-tables-rows-and-columns)
4. [SELECT Statement](#4-select-statement)
5. [FROM Clause](#5-from-clause)
6. [WHERE Clause](#6-where-clause)
7. [DISTINCT Keyword](#7-distinct-keyword)
8. [ORDER BY Clause](#8-order-by-clause)
9. [LIMIT and TOP](#9-limit-and-top)
10. [Comparison Operators](#10-comparison-operators)
11. [Logical Operators](#11-logical-operators)
12. [BETWEEN Operator](#12-between-operator)
13. [IN Operator](#13-in-operator)
14. [LIKE Operator](#14-like-operator)
15. [NULL Handling](#15-null-handling)

---

## 1. What is SQL?

### Theory
**SQL** stands for **Structured Query Language**. It is a standardized programming language specifically designed for managing and manipulating data stored in relational database management systems (RDBMS).

**Key Points:**
- SQL is declarative - you specify *what* you want, not *how* to get it
- Used for querying, inserting, updating, and deleting data
- Also used for creating and modifying database structures
- Not case-sensitive (though convention uses uppercase for keywords)
- Statements typically end with a semicolon (;)

**Purpose:**
- Retrieve specific data from databases
- Perform operations on data (add, modify, delete)
- Create and manage database structures
- Control access to data

---

## 2. RDBMS Concepts

### Theory
**RDBMS** stands for **Relational Database Management System**. It's a software system that manages databases based on the relational model.

**Core Principles:**

**1. Relational Model:**
- Data is organized into tables (relations)
- Each table represents an entity (e.g., customers, products, orders)
- Relationships exist between tables

**2. Key Characteristics:**
- **Data Integrity:** Ensures accuracy and consistency
- **ACID Properties:**
  - **Atomicity:** Transactions complete fully or not at all
  - **Consistency:** Data remains valid after transactions
  - **Isolation:** Concurrent transactions don't interfere
  - **Durability:** Completed transactions are permanent

**3. Common RDBMS:**
- MySQL
- PostgreSQL
- Oracle Database
- Microsoft SQL Server
- SQLite

**4. Schema:**
- Defines the structure of the database
- Specifies tables, columns, data types, and relationships
- Acts as a blueprint for the database

---

## 3. Tables, Rows, and Columns

### Theory

**Table (Relation):**
- A collection of related data organized in a structured format
- Represents a specific entity type (e.g., Employees, Products)
- Has a unique name within the database

**Columns (Attributes/Fields):**
- Vertical structures in a table
- Represent specific characteristics of the entity
- Each column has:
  - **Name:** Unique identifier within the table
  - **Data Type:** Defines what kind of data can be stored (INT, VARCHAR, DATE, etc.)
  - **Constraints:** Rules like NOT NULL, UNIQUE, PRIMARY KEY

**Rows (Records/Tuples):**
- Horizontal structures in a table
- Represent individual instances of the entity
- Each row contains values for all columns

**Visual Representation:**
```
Table: employees
┌────┬──────────┬───────────┬────────┐
│ id │   name   │ department│ salary │  ← Column Names
├────┼──────────┼───────────┼────────┤
│ 1  │ John     │ Sales     │ 50000  │  ← Row 1
│ 2  │ Sarah    │ IT        │ 60000  │  ← Row 2
│ 3  │ Mike     │ Sales     │ 45000  │  ← Row 3
└────┴──────────┴───────────┴────────┘
```

**Primary Key:**
- A column (or set of columns) that uniquely identifies each row
- Cannot contain NULL values
- Must be unique for each row
- Example: `id` column in the employees table

---

## 4. SELECT Statement

### Theory
The `SELECT` statement is the most fundamental SQL command used to retrieve data from a database. It specifies which columns you want to retrieve.

**Syntax:**
```sql
SELECT column1, column2, ...
FROM table_name;
```

**Key Concepts:**

**1. Select Specific Columns:**
- List the exact columns you want
- Columns are returned in the order specified

**2. Select All Columns:**
- Use asterisk (*) to select all columns
- Convenient but less efficient for large tables

**3. Column Aliases:**
- Rename columns in the output using `AS`
- Makes results more readable

**4. Expressions:**
- Can perform calculations in SELECT
- Combine columns or use functions

### Examples

```sql
-- Select specific columns
SELECT name, salary
FROM employees;

-- Select all columns
SELECT *
FROM employees;

-- Using column aliases
SELECT name AS employee_name, salary AS annual_salary
FROM employees;

-- Expressions in SELECT
SELECT name, salary * 12 AS yearly_salary
FROM employees;
```

---

## 5. FROM Clause

### Theory
The `FROM` clause specifies the table(s) from which to retrieve data. It tells SQL where to look for the data you're selecting.

**Key Points:**
- Mandatory in most SELECT statements (except when selecting constants)
- Follows the SELECT clause
- Can reference one table or multiple tables (joins)
- Table names must exist in the database

**Syntax:**
```sql
SELECT column_list
FROM table_name;
```

**Multiple Tables:**
- Later you'll learn to combine data from multiple tables
- Uses JOIN operations
- FROM can list several tables with relationships

### Examples

```sql
-- Basic FROM usage
SELECT name, department
FROM employees;

-- Selecting from a different table
SELECT product_name, price
FROM products;

-- Schema-qualified table name (specifying database/schema)
SELECT name
FROM company.employees;
```

---

## 6. WHERE Clause

### Theory
The `WHERE` clause filters rows based on specified conditions. It acts as a filter that determines which rows to include in the result set.

**Key Concepts:**

**1. Purpose:**
- Reduces the result set to only rows that meet criteria
- Applied before data is returned
- Can combine multiple conditions

**2. Execution Order:**
- WHERE is processed after FROM
- Before SELECT (conceptually)
- Filters first, then retrieves specified columns

**3. Conditions:**
- Must evaluate to TRUE, FALSE, or NULL
- Only rows where condition is TRUE are returned

**Syntax:**
```sql
SELECT column_list
FROM table_name
WHERE condition;
```

### Examples

```sql
-- Simple condition
SELECT name, salary
FROM employees
WHERE department = 'Sales';

-- Numeric comparison
SELECT name, salary
FROM employees
WHERE salary > 50000;

-- Multiple conditions (covered more in Logical Operators)
SELECT name, department
FROM employees
WHERE department = 'IT' AND salary > 55000;
```

---

## 7. DISTINCT Keyword

### Theory
The `DISTINCT` keyword eliminates duplicate rows from the result set. It returns only unique combinations of the specified columns.

**Key Points:**

**1. Purpose:**
- Remove duplicate rows from results
- Show unique values only
- Useful for getting lists of unique categories, departments, etc.

**2. Behavior:**
- Compares entire rows (all selected columns together)
- NULL values are considered equal to each other
- Can impact performance on large datasets

**3. Position:**
- Placed immediately after SELECT
- Applies to all columns in the SELECT list

**Syntax:**
```sql
SELECT DISTINCT column_list
FROM table_name;
```

**Important Note:**
When using DISTINCT with multiple columns, it returns unique *combinations* of those columns, not unique values for each column independently.

### Examples

```sql
-- Get unique departments
SELECT DISTINCT department
FROM employees;

-- Get unique combinations of department and location
SELECT DISTINCT department, location
FROM employees;

-- Without DISTINCT (shows duplicates)
SELECT department
FROM employees;
-- Result might be: Sales, IT, Sales, Sales, IT

-- With DISTINCT (shows unique values)
SELECT DISTINCT department
FROM employees;
-- Result: Sales, IT
```

---

## 8. ORDER BY Clause

### Theory
The `ORDER BY` clause sorts the result set based on one or more columns. It determines the sequence in which rows are returned.

**Key Concepts:**

**1. Sort Direction:**
- **ASC:** Ascending order (default, lowest to highest)
- **DESC:** Descending order (highest to lowest)

**2. Multiple Columns:**
- Can sort by multiple columns in priority order
- First column is primary sort, second is tiebreaker, etc.

**3. Execution Order:**
- Applied after WHERE clause
- One of the last operations performed
- Does not affect which rows are returned, only their order

**4. Sorting Rules:**
- Numbers: Numeric order (-1, 0, 1, 2, 10, 100)
- Strings: Alphabetical order (A-Z)
- Dates: Chronological order
- NULL values: Typically appear first or last depending on RDBMS

**Syntax:**
```sql
SELECT column_list
FROM table_name
WHERE condition
ORDER BY column1 [ASC|DESC], column2 [ASC|DESC];
```

### Examples

```sql
-- Sort by salary ascending (low to high)
SELECT name, salary
FROM employees
ORDER BY salary ASC;

-- Sort by salary descending (high to low)
SELECT name, salary
FROM employees
ORDER BY salary DESC;

-- Multiple columns: sort by department, then salary within department
SELECT name, department, salary
FROM employees
ORDER BY department ASC, salary DESC;

-- Sort by column position (1 = first column in SELECT)
SELECT name, salary
FROM employees
ORDER BY 2 DESC;  -- Sorts by salary (2nd column)
```

---

## 9. LIMIT and TOP

### Theory
`LIMIT` and `TOP` restrict the number of rows returned by a query. Different RDBMS systems use different syntax for this functionality.

**Key Points:**

**1. Purpose:**
- Retrieve only a specific number of rows
- Useful for pagination
- Helps with performance on large datasets
- Get top N results

**2. RDBMS Variations:**
- **MySQL, PostgreSQL:** Use `LIMIT`
- **SQL Server, MS Access:** Use `TOP`
- **Oracle:** Use `ROWNUM` or `FETCH FIRST`

**3. Combination with ORDER BY:**
- Often used together to get "top N" results
- Without ORDER BY, results are arbitrary
- ORDER BY determines which rows are "first"

**4. OFFSET (Pagination):**
- Some systems support OFFSET with LIMIT
- Allows skipping rows for pagination
- Syntax: `LIMIT count OFFSET skip`

### Syntax

```sql
-- MySQL/PostgreSQL
SELECT column_list
FROM table_name
LIMIT number;

-- SQL Server
SELECT TOP number column_list
FROM table_name;

-- With OFFSET (MySQL/PostgreSQL)
SELECT column_list
FROM table_name
LIMIT number OFFSET skip;
```

### Examples

```sql
-- Get first 5 employees (MySQL/PostgreSQL)
SELECT name, salary
FROM employees
LIMIT 5;

-- Get first 5 employees (SQL Server)
SELECT TOP 5 name, salary
FROM employees;

-- Get top 3 highest paid employees
SELECT name, salary
FROM employees
ORDER BY salary DESC
LIMIT 3;

-- Pagination: Skip first 10, get next 5
SELECT name, salary
FROM employees
ORDER BY name
LIMIT 5 OFFSET 10;

-- Get 10-20% of results
SELECT name, salary
FROM employees
LIMIT 10 OFFSET 0;  -- Page 1

SELECT name, salary
FROM employees
LIMIT 10 OFFSET 10;  -- Page 2
```

---

## 10. Comparison Operators

### Theory
Comparison operators compare two values and return TRUE, FALSE, or NULL. They are essential for filtering data in WHERE clauses.

**Common Comparison Operators:**

| Operator | Meaning | Example |
|----------|---------|---------|
| `=` | Equal to | `salary = 50000` |
| `!=` or `<>` | Not equal to | `department != 'Sales'` |
| `>` | Greater than | `salary > 50000` |
| `<` | Less than | `age < 30` |
| `>=` | Greater than or equal | `salary >= 50000` |
| `<=` | Less than or equal | `age <= 65` |

**Key Points:**

**1. Data Type Compatibility:**
- Both operands should be compatible types
- Comparing numbers to numbers, strings to strings, etc.
- Type conversion may occur automatically

**2. String Comparison:**
- Usually case-sensitive (depends on database collation)
- Alphabetical order: 'A' < 'B' < 'Z'
- 'Apple' < 'Banana'

**3. Date Comparison:**
- Can compare dates chronologically
- '2024-01-01' < '2024-12-31'

**4. NULL Considerations:**
- NULL compared to any value (including NULL) returns NULL
- Use special NULL handling operators (covered later)

### Examples

```sql
-- Equal to
SELECT name, salary
FROM employees
WHERE salary = 50000;

-- Not equal to (both syntaxes work)
SELECT name, department
FROM employees
WHERE department != 'Sales';

SELECT name, department
FROM employees
WHERE department <> 'Sales';

-- Greater than
SELECT name, salary
FROM employees
WHERE salary > 60000;

-- Less than or equal
SELECT name, age
FROM employees
WHERE age <= 30;

-- String comparison
SELECT name, city
FROM employees
WHERE city = 'New York';

-- Date comparison
SELECT name, hire_date
FROM employees
WHERE hire_date >= '2023-01-01';
```

---

## 11. Logical Operators

### Theory
Logical operators combine multiple conditions in WHERE clauses. They allow complex filtering based on multiple criteria.

**Three Main Logical Operators:**

**1. AND Operator:**
- Returns TRUE only if ALL conditions are TRUE
- Narrows down results (more restrictive)
- Truth table:
  - TRUE AND TRUE = TRUE
  - TRUE AND FALSE = FALSE
  - FALSE AND FALSE = FALSE

**2. OR Operator:**
- Returns TRUE if ANY condition is TRUE
- Broadens results (less restrictive)
- Truth table:
  - TRUE OR TRUE = TRUE
  - TRUE OR FALSE = TRUE
  - FALSE OR FALSE = FALSE

**3. NOT Operator:**
- Reverses the truth value
- Negates a condition
- Truth table:
  - NOT TRUE = FALSE
  - NOT FALSE = TRUE

**Precedence:**
- NOT has highest precedence
- AND has higher precedence than OR
- Use parentheses () to control evaluation order

**Combining Operators:**
- Can combine multiple ANDs, ORs, and NOTs
- Parentheses clarify complex logic

### Syntax

```sql
-- AND
SELECT column_list
FROM table_name
WHERE condition1 AND condition2;

-- OR
SELECT column_list
FROM table_name
WHERE condition1 OR condition2;

-- NOT
SELECT column_list
FROM table_name
WHERE NOT condition;
```

### Examples

```sql
-- AND: Both conditions must be true
SELECT name, department, salary
FROM employees
WHERE department = 'Sales' AND salary > 45000;

-- OR: Either condition can be true
SELECT name, department
FROM employees
WHERE department = 'Sales' OR department = 'IT';

-- NOT: Negates the condition
SELECT name, department
FROM employees
WHERE NOT department = 'Sales';

-- Complex combination with parentheses
SELECT name, department, salary
FROM employees
WHERE (department = 'Sales' OR department = 'IT')
  AND salary > 50000;

-- Multiple AND conditions
SELECT name, age, salary, department
FROM employees
WHERE age >= 25
  AND age <= 40
  AND salary > 45000
  AND department = 'IT';

-- Combining all three
SELECT name, department, salary
FROM employees
WHERE (department = 'Sales' OR department = 'IT')
  AND NOT salary < 40000;
```

---

## 12. BETWEEN Operator

### Theory
The `BETWEEN` operator selects values within a specified range. It's a shorthand for checking if a value falls between two boundary values.

**Key Points:**

**1. Inclusive Range:**
- Includes both boundary values
- `x BETWEEN a AND b` means `x >= a AND x <= b`

**2. Equivalent Expression:**
```sql
-- These are equivalent:
WHERE salary BETWEEN 40000 AND 60000
WHERE salary >= 40000 AND salary <= 60000
```

**3. Data Types:**
- Works with numbers, dates, and text
- Boundaries must be the same type as the column

**4. NOT BETWEEN:**
- Excludes the specified range
- `x NOT BETWEEN a AND b` means `x < a OR x > b`

**5. Order Matters:**
- Lower boundary comes first
- `BETWEEN 40000 AND 60000` (correct)
- `BETWEEN 60000 AND 40000` (incorrect - returns no results)

### Syntax

```sql
SELECT column_list
FROM table_name
WHERE column_name BETWEEN value1 AND value2;

-- Negated form
SELECT column_list
FROM table_name
WHERE column_name NOT BETWEEN value1 AND value2;
```

### Examples

```sql
-- Numeric range: salaries between 40,000 and 60,000
SELECT name, salary
FROM employees
WHERE salary BETWEEN 40000 AND 60000;

-- Date range: hired in 2023
SELECT name, hire_date
FROM employees
WHERE hire_date BETWEEN '2023-01-01' AND '2023-12-31';

-- NOT BETWEEN: exclude a range
SELECT name, age
FROM employees
WHERE age NOT BETWEEN 30 AND 50;

-- Text range (alphabetical)
SELECT name
FROM employees
WHERE name BETWEEN 'A' AND 'M';

-- Combining with other conditions
SELECT name, salary, department
FROM employees
WHERE salary BETWEEN 45000 AND 65000
  AND department = 'Sales';
```

---

## 13. IN Operator

### Theory
The `IN` operator checks if a value matches any value in a list. It's a cleaner alternative to multiple OR conditions.

**Key Points:**

**1. Purpose:**
- Test if a value exists in a set of values
- Simplifies multiple OR comparisons
- More readable than chained OR statements

**2. Equivalent Expression:**
```sql
-- These are equivalent:
WHERE department IN ('Sales', 'IT', 'Marketing')
WHERE department = 'Sales' OR department = 'IT' OR department = 'Marketing'
```

**3. NOT IN:**
- Excludes values in the list
- Returns rows where value doesn't match any list item

**4. Value List:**
- Can be literal values
- Can be a subquery (returns values from another query)
- All values must be compatible with the column type

**5. NULL Handling:**
- If the list contains NULL, behavior varies
- Generally safer to use explicit NULL checks

### Syntax

```sql
SELECT column_list
FROM table_name
WHERE column_name IN (value1, value2, value3, ...);

-- Negated form
SELECT column_list
FROM table_name
WHERE column_name NOT IN (value1, value2, value3, ...);
```

### Examples

```sql
-- Check if department is in a list
SELECT name, department
FROM employees
WHERE department IN ('Sales', 'IT', 'Marketing');

-- Numeric values
SELECT name, employee_id
FROM employees
WHERE employee_id IN (101, 105, 110, 115);

-- NOT IN: exclude specific values
SELECT name, department
FROM employees
WHERE department NOT IN ('Sales', 'HR');

-- Combining with AND
SELECT name, department, salary
FROM employees
WHERE department IN ('Sales', 'IT')
  AND salary > 50000;

-- Single value (works but unusual)
SELECT name
FROM employees
WHERE department IN ('Sales');
-- Same as: WHERE department = 'Sales'
```

---

## 14. LIKE Operator

### Theory
The `LIKE` operator performs pattern matching on text strings. It's used for searching text that matches a specific pattern rather than exact values.

**Wildcard Characters:**

**1. Percent Sign (%):**
- Represents zero, one, or multiple characters
- Can be placed anywhere in the pattern

**2. Underscore (_):**
- Represents exactly one character
- Used for patterns with known length

**Common Patterns:**

| Pattern | Meaning | Example Match |
|---------|---------|---------------|
| `'A%'` | Starts with A | Apple, Ant, Avocado |
| `'%a'` | Ends with a | Pizza, Data, Java |
| `'%or%'` | Contains 'or' | Morning, World, Orange |
| `'_at'` | Three letters ending in 'at' | Cat, Bat, Hat |
| `'A__%'` | Starts with A, at least 3 chars | Apple, About |

**Key Points:**

**1. Case Sensitivity:**
- Depends on database collation
- MySQL: Usually case-insensitive
- PostgreSQL: Case-sensitive by default (use ILIKE for case-insensitive)

**2. NOT LIKE:**
- Excludes patterns
- Returns rows that don't match the pattern

**3. Escaping Wildcards:**
- Use ESCAPE clause if you need to search for literal % or _
- Example: `WHERE text LIKE '%\%%' ESCAPE '\'`

**4. Performance:**
- Leading wildcards (`%text`) prevent index usage
- Can be slow on large datasets

### Syntax

```sql
SELECT column_list
FROM table_name
WHERE column_name LIKE 'pattern';

-- Negated form
SELECT column_list
FROM table_name
WHERE column_name NOT LIKE 'pattern';
```

### Examples

```sql
-- Starts with 'J'
SELECT name
FROM employees
WHERE name LIKE 'J%';
-- Matches: John, Jane, James

-- Ends with 'son'
SELECT name
FROM employees
WHERE name LIKE '%son';
-- Matches: Johnson, Anderson, Wilson

-- Contains 'an'
SELECT name
FROM employees
WHERE name LIKE '%an%';
-- Matches: Anderson, Daniel, Hannah

-- Exactly 4 characters
SELECT name
FROM employees
WHERE name LIKE '____';
-- Matches: John, Mike, Sara

-- Second character is 'a'
SELECT name
FROM employees
WHERE name LIKE '_a%';
-- Matches: Jane, Sarah, James

-- NOT LIKE: exclude pattern
SELECT name
FROM employees
WHERE name NOT LIKE 'J%';
-- Excludes names starting with J

-- Combining wildcards
SELECT email
FROM employees
WHERE email LIKE '%@gmail.com';
-- Matches Gmail addresses

-- Multiple patterns with OR
SELECT name
FROM employees
WHERE name LIKE 'J%' OR name LIKE 'S%';
-- Matches names starting with J or S
```

---

## 15. NULL Handling

### Theory
`NULL` represents the absence of a value. It's not zero, not an empty string, but specifically "no value" or "unknown value". NULL requires special handling in SQL.

**Key Concepts:**

**1. What is NULL:**
- Represents missing, unknown, or inapplicable data
- Different from zero (0) or empty string ('')
- Has special comparison rules

**2. NULL Comparison Behavior:**
- NULL compared to anything (including NULL) results in NULL
- NULL is neither TRUE nor FALSE
- `NULL = NULL` returns NULL (not TRUE!)
- `NULL != NULL` returns NULL (not FALSE!)

**3. Three-Valued Logic:**
SQL uses three-valued logic: TRUE, FALSE, and NULL (unknown)
- TRUE AND NULL = NULL
- TRUE OR NULL = TRUE
- FALSE OR NULL = NULL

**Special NULL Operators:**

**IS NULL:**
- Tests if a value is NULL
- Returns TRUE if value is NULL

**IS NOT NULL:**
- Tests if a value is NOT NULL
- Returns TRUE if value exists

**COALESCE:**
- Returns first non-NULL value from a list
- Useful for providing default values

**NULLIF:**
- Returns NULL if two values are equal
- Otherwise returns the first value

### Why Normal Operators Don't Work with NULL

```sql
-- WRONG: These don't work as expected
WHERE column = NULL     -- Always returns NULL (no rows)
WHERE column != NULL    -- Always returns NULL (no rows)

-- CORRECT: Use IS NULL
WHERE column IS NULL
WHERE column IS NOT NULL
```

### Syntax

```sql
-- Check for NULL
SELECT column_list
FROM table_name
WHERE column_name IS NULL;

-- Check for NOT NULL
SELECT column_list
FROM table_name
WHERE column_name IS NOT NULL;

-- COALESCE (return first non-NULL)
SELECT COALESCE(column1, column2, 'default_value')
FROM table_name;
```

### Examples

```sql
-- Find employees with no phone number
SELECT name, phone
FROM employees
WHERE phone IS NULL;

-- Find employees with a phone number
SELECT name, phone
FROM employees
WHERE phone IS NOT NULL;

-- Exclude rows with NULL salary
SELECT name, salary
FROM employees
WHERE salary IS NOT NULL;

-- COALESCE: provide default value
SELECT name, COALESCE(phone, 'No phone') AS contact
FROM employees;

-- COALESCE: use first available value
SELECT name, COALESCE(mobile, office_phone, 'No contact') AS phone
FROM employees;

-- Combining with other conditions
SELECT name, department, phone
FROM employees
WHERE department = 'Sales'
  AND phone IS NOT NULL;

-- NULL in calculations (NULL propagates)
SELECT name, salary, salary * 1.1 AS raised_salary
FROM employees;
-- If salary is NULL, raised_salary will also be NULL

-- Handle NULL in calculations
SELECT name, 
       COALESCE(salary, 0) * 1.1 AS raised_salary
FROM employees;

-- NULLIF example: return NULL if values match
SELECT name,
       NULLIF(department, 'Unknown') AS dept
FROM employees;
-- Returns NULL for 'Unknown' departments
```

### NULL in Different Contexts

```sql
-- COUNT ignores NULL values
SELECT COUNT(phone) FROM employees;  -- Counts non-NULL phones
SELECT COUNT(*) FROM employees;      -- Counts all rows

-- NULL with AND/OR
SELECT name
FROM employees
WHERE salary > 50000 AND bonus IS NULL;

-- ORDER BY with NULL
SELECT name, bonus
FROM employees
ORDER BY bonus;  -- NULLs appear first or last (RDBMS-dependent)

-- NULL-safe equality (MySQL specific)
SELECT name
FROM employees
WHERE column <=> NULL;  -- MySQL only
```

---

## Query Execution Order (Conceptual)

Understanding the logical order in which SQL processes clauses helps write better queries:

```
1. FROM       - Identify the table(s)
2. WHERE      - Filter rows
3. SELECT     - Choose columns
4. DISTINCT   - Remove duplicates
5. ORDER BY   - Sort results
6. LIMIT/TOP  - Restrict number of rows
```

**Note:** This is the *logical* order. The database optimizer may execute in a different physical order for efficiency.

---

## Best Practices

1. **Use specific column names** instead of `SELECT *` when possible
2. **Always use table names** when querying multiple tables
3. **Use parentheses** to clarify complex logical conditions
4. **Handle NULLs explicitly** with IS NULL / IS NOT NULL
5. **Use BETWEEN** instead of `>= AND <=` for readability
6. **Use IN** instead of multiple ORs for cleaner code
7. **Add ORDER BY** when using LIMIT/TOP for predictable results
8. **Comment complex queries** to explain business logic
9. **Test NULL handling** in WHERE clauses
10. **Use meaningful aliases** for better readability

---

## Common Mistakes to Avoid

1. **Using `= NULL`** instead of `IS NULL`
2. **Forgetting ORDER BY** with LIMIT (results may vary)
3. **Mixing up AND/OR precedence** without parentheses
4. **Using DISTINCT unnecessarily** (performance impact)
5. **Reversed BETWEEN bounds** (40000, 30000 instead of 30000, 40000)
6. **Case sensitivity issues** with LIKE operator
7. **Assuming NULL equals empty string** or zero
8. **Not considering NULL** in comparison operations
9. **Using leading wildcards** (`%term`) without performance considerations
10. **Confusing `!=` behavior** with NULL values

---

## Practice Exercise Structure

To master these fundamentals, practice queries in this order:

1. Simple SELECT with specific columns
2. Add WHERE with single condition
3. Add multiple WHERE conditions with AND/OR
4. Use BETWEEN, IN, LIKE for pattern matching
5. Add ORDER BY for sorting
6. Use DISTINCT to find unique values
7. Add LIMIT/TOP for result restriction
8. Practice NULL handling in various scenarios
9. Combine all concepts in complex queries

---

## Summary

SQL Fundamentals provide the foundation for all database work:

- **SELECT** retrieves data from columns
- **FROM** specifies the source table
- **WHERE** filters rows based on conditions
- **DISTINCT** removes duplicate rows
- **ORDER BY** sorts results
- **LIMIT/TOP** restricts row count
- **Comparison operators** (=, !=, >, <, >=, <=) compare values
- **Logical operators** (AND, OR, NOT) combine conditions
- **BETWEEN** checks ranges inclusively
- **IN** matches values in a list
- **LIKE** performs pattern matching with wildcards
- **NULL** requires special handling with IS NULL/IS NOT NULL

Master these concepts thoroughly—they form the building blocks for advanced SQL topics like JOINs, subqueries, aggregations, and more.

---

**End of SQL Fundamentals Guide**
