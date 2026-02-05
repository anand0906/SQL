# SQL Aggregations & Grouping - Complete Guide

## Table of Contents
1. [Introduction to Aggregations](#1-introduction-to-aggregations)
2. [Sample Tables for Examples](#2-sample-tables-for-examples)
3. [COUNT Function](#3-count-function)
4. [SUM Function](#4-sum-function)
5. [AVG Function](#5-avg-function)
6. [MIN Function](#6-min-function)
7. [MAX Function](#7-max-function)
8. [GROUP BY Clause](#8-group-by-clause)
9. [HAVING Clause](#9-having-clause)
10. [WHERE vs HAVING](#10-where-vs-having)
11. [Combining Aggregations](#11-combining-aggregations)
12. [Common Patterns](#12-common-patterns)
13. [Best Practices](#13-best-practices)

---

## 1. Introduction to Aggregations

### Theory

**Aggregate Functions** perform calculations on a set of rows and return a single value. They are used to summarize data.

**Key Concepts:**

**1. Purpose:**
- Summarize multiple rows into a single result
- Calculate statistics (totals, averages, counts)
- Answer questions like "How many?", "What's the total?", "What's the average?"

**2. Common Uses:**
- Business reporting (total sales, average salary)
- Data analysis (finding trends, patterns)
- Statistical summaries

**3. Important Rules:**
- Aggregate functions ignore NULL values (except COUNT(*))
- Cannot mix aggregated and non-aggregated columns without GROUP BY
- Can be used in SELECT and HAVING clauses

**4. The Five Core Aggregate Functions:**
- **COUNT**: Counts rows or non-NULL values
- **SUM**: Adds up numeric values
- **AVG**: Calculates average of numeric values
- **MIN**: Finds minimum value
- **MAX**: Finds maximum value

---

## 2. Sample Tables for Examples

We'll use these simple tables throughout the guide:

### Table: employees
```
┌────┬──────────┬────────────┬────────┬─────────┐
│ id │   name   │ department │ salary │ manager │
├────┼──────────┼────────────┼────────┼─────────┤
│ 1  │ John     │ Sales      │ 50000  │ Alice   │
│ 2  │ Sarah    │ IT         │ 60000  │ Bob     │
│ 3  │ Mike     │ Sales      │ 45000  │ Alice   │
│ 4  │ Emma     │ IT         │ 65000  │ Bob     │
│ 5  │ David    │ HR         │ 55000  │ NULL    │
└────┴──────────┴────────────┴────────┴─────────┘
```

### Table: orders
```
┌──────────┬─────────────┬────────┬──────────┐
│ order_id │ customer_id │ amount │  status  │
├──────────┼─────────────┼────────┼──────────┤
│ 101      │ 1           │ 250    │ Shipped  │
│ 102      │ 2           │ 150    │ Pending  │
│ 103      │ 1           │ 300    │ Shipped  │
│ 104      │ 3           │ 200    │ Shipped  │
│ 105      │ 2           │ 100    │ Canceled │
└──────────┴─────────────┴────────┴──────────┘
```

### Table: products
```
┌────────────┬──────────┬───────┬─────────────┐
│ product_id │   name   │ price │  category   │
├────────────┼──────────┼───────┼─────────────┤
│ 1          │ Laptop   │ 1200  │ Electronics │
│ 2          │ Mouse    │ 25    │ Electronics │
│ 3          │ Desk     │ 300   │ Furniture   │
│ 4          │ Chair    │ 150   │ Furniture   │
│ 5          │ Monitor  │ 400   │ Electronics │
└────────────┴──────────┴───────┴─────────────┘
```

---

## 3. COUNT Function

### Theory

**COUNT** returns the number of rows that match a criteria or the number of non-NULL values in a column.

**Three Variants:**

**1. COUNT(*):**
- Counts all rows including those with NULL values
- Most commonly used
- Counts the total number of records

**2. COUNT(column_name):**
- Counts non-NULL values in the specified column
- Ignores NULL values
- Useful for checking data completeness

**3. COUNT(DISTINCT column_name):**
- Counts unique non-NULL values
- Removes duplicates before counting

**Key Points:**
- Returns integer value
- Never returns NULL (returns 0 if no rows match)
- Can be combined with WHERE for conditional counting

### Syntax
```sql
COUNT(*)
COUNT(column_name)
COUNT(DISTINCT column_name)
```

### Examples with Output

**Example 1: Count all employees**
```sql
SELECT COUNT(*) AS total_employees
FROM employees;
```

**Output:**
```
┌──────────────────┐
│ total_employees  │
├──────────────────┤
│ 5                │
└──────────────────┘
```

---

**Example 2: Count employees with managers**
```sql
SELECT COUNT(manager) AS employees_with_manager
FROM employees;
```

**Output:**
```
┌─────────────────────────┐
│ employees_with_manager  │
├─────────────────────────┤
│ 4                       │
└─────────────────────────┘
```
*Note: David has NULL manager, so only 4 are counted*

---

**Example 3: Count distinct departments**
```sql
SELECT COUNT(DISTINCT department) AS unique_departments
FROM employees;
```

**Output:**
```
┌─────────────────────┐
│ unique_departments  │
├─────────────────────┤
│ 3                   │
└─────────────────────┘
```
*Note: Sales, IT, HR = 3 unique departments*

---

**Example 4: Count with WHERE condition**
```sql
SELECT COUNT(*) AS it_employees
FROM employees
WHERE department = 'IT';
```

**Output:**
```
┌───────────────┐
│ it_employees  │
├───────────────┤
│ 2             │
└───────────────┘
```

---

## 4. SUM Function

### Theory

**SUM** calculates the total of numeric values in a column.

**Key Points:**
- Only works with numeric data types (INT, DECIMAL, FLOAT, etc.)
- Ignores NULL values
- Returns NULL if all values are NULL
- Can cause overflow with very large numbers
- Often used for financial calculations (total sales, revenue)

**Common Uses:**
- Calculate total sales
- Sum of salaries
- Total inventory value
- Aggregate monetary amounts

### Syntax
```sql
SUM(column_name)
```

### Examples with Output

**Example 1: Total salary expense**
```sql
SELECT SUM(salary) AS total_salary_expense
FROM employees;
```

**Output:**
```
┌───────────────────────┐
│ total_salary_expense  │
├───────────────────────┤
│ 275000                │
└───────────────────────┘
```
*Calculation: 50000 + 60000 + 45000 + 65000 + 55000 = 275000*

---

**Example 2: Total order amount**
```sql
SELECT SUM(amount) AS total_order_value
FROM orders;
```

**Output:**
```
┌────────────────────┐
│ total_order_value  │
├────────────────────┤
│ 1000               │
└────────────────────┘
```
*Calculation: 250 + 150 + 300 + 200 + 100 = 1000*

---

**Example 3: Sum with WHERE condition**
```sql
SELECT SUM(amount) AS shipped_orders_total
FROM orders
WHERE status = 'Shipped';
```

**Output:**
```
┌───────────────────────┐
│ shipped_orders_total  │
├───────────────────────┤
│ 750                   │
└───────────────────────┘
```
*Calculation: 250 + 300 + 200 = 750 (only Shipped orders)*

---

**Example 4: Sum of salaries in IT department**
```sql
SELECT SUM(salary) AS it_department_payroll
FROM employees
WHERE department = 'IT';
```

**Output:**
```
┌────────────────────────┐
│ it_department_payroll  │
├────────────────────────┤
│ 125000                 │
└────────────────────────┘
```
*Calculation: 60000 + 65000 = 125000*

---

## 5. AVG Function

### Theory

**AVG** calculates the average (arithmetic mean) of numeric values in a column.

**Key Points:**
- Only works with numeric data types
- Ignores NULL values in calculation
- Returns NULL if all values are NULL
- Formula: SUM(values) / COUNT(non-NULL values)
- May return decimal/float even if input is integer

**Common Uses:**
- Average salary
- Mean order value
- Average rating
- Statistical analysis

**Important:** AVG ignores NULLs, which affects the denominator:
- If you have values [10, 20, NULL], AVG = 15 (not 10)
- It's (10 + 20) / 2, not (10 + 20 + 0) / 3

### Syntax
```sql
AVG(column_name)
```

### Examples with Output

**Example 1: Average salary**
```sql
SELECT AVG(salary) AS average_salary
FROM employees;
```

**Output:**
```
┌─────────────────┐
│ average_salary  │
├─────────────────┤
│ 55000.00        │
└─────────────────┘
```
*Calculation: (50000 + 60000 + 45000 + 65000 + 55000) / 5 = 55000*

---

**Example 2: Average order amount**
```sql
SELECT AVG(amount) AS average_order_value
FROM orders;
```

**Output:**
```
┌─────────────────────┐
│ average_order_value │
├─────────────────────┤
│ 200.00              │
└─────────────────────┘
```
*Calculation: (250 + 150 + 300 + 200 + 100) / 5 = 200*

---

**Example 3: Average salary in Sales department**
```sql
SELECT AVG(salary) AS avg_sales_salary
FROM employees
WHERE department = 'Sales';
```

**Output:**
```
┌───────────────────┐
│ avg_sales_salary  │
├───────────────────┤
│ 47500.00          │
└───────────────────┘
```
*Calculation: (50000 + 45000) / 2 = 47500*

---

**Example 4: Average price of Electronics**
```sql
SELECT AVG(price) AS avg_electronics_price
FROM products
WHERE category = 'Electronics';
```

**Output:**
```
┌────────────────────────┐
│ avg_electronics_price  │
├────────────────────────┤
│ 541.67                 │
└────────────────────────┘
```
*Calculation: (1200 + 25 + 400) / 3 = 541.67*

---

## 6. MIN Function

### Theory

**MIN** returns the smallest value in a column.

**Key Points:**
- Works with numeric, date, and text data
- Ignores NULL values
- Returns NULL if all values are NULL
- For numbers: finds smallest number
- For text: finds alphabetically first value (A before Z)
- For dates: finds earliest date

**Common Uses:**
- Lowest salary
- Earliest date
- Minimum price
- First alphabetically

### Syntax
```sql
MIN(column_name)
```

### Examples with Output

**Example 1: Lowest salary**
```sql
SELECT MIN(salary) AS lowest_salary
FROM employees;
```

**Output:**
```
┌───────────────┐
│ lowest_salary │
├───────────────┤
│ 45000         │
└───────────────┘
```

---

**Example 2: Cheapest product**
```sql
SELECT MIN(price) AS cheapest_price
FROM products;
```

**Output:**
```
┌────────────────┐
│ cheapest_price │
├────────────────┤
│ 25             │
└────────────────┘
```

---

**Example 3: Minimum order amount**
```sql
SELECT MIN(amount) AS smallest_order
FROM orders;
```

**Output:**
```
┌────────────────┐
│ smallest_order │
├────────────────┤
│ 100            │
└────────────────┘
```

---

**Example 4: First name alphabetically**
```sql
SELECT MIN(name) AS first_name_alphabetically
FROM employees;
```

**Output:**
```
┌───────────────────────────┐
│ first_name_alphabetically │
├───────────────────────────┤
│ David                     │
└───────────────────────────┘
```
*D comes before E, J, M, S alphabetically*

---

## 7. MAX Function

### Theory

**MAX** returns the largest value in a column.

**Key Points:**
- Works with numeric, date, and text data
- Ignores NULL values
- Returns NULL if all values are NULL
- For numbers: finds largest number
- For text: finds alphabetically last value (Z after A)
- For dates: finds most recent date

**Common Uses:**
- Highest salary
- Latest date
- Maximum price
- Last alphabetically

### Syntax
```sql
MAX(column_name)
```

### Examples with Output

**Example 1: Highest salary**
```sql
SELECT MAX(salary) AS highest_salary
FROM employees;
```

**Output:**
```
┌────────────────┐
│ highest_salary │
├────────────────┤
│ 65000          │
└────────────────┘
```

---

**Example 2: Most expensive product**
```sql
SELECT MAX(price) AS most_expensive
FROM products;
```

**Output:**
```
┌─────────────────┐
│ most_expensive  │
├─────────────────┤
│ 1200            │
└─────────────────┘
```

---

**Example 3: Largest order amount**
```sql
SELECT MAX(amount) AS largest_order
FROM orders;
```

**Output:**
```
┌───────────────┐
│ largest_order │
├───────────────┤
│ 300           │
└───────────────┘
```

---

**Example 4: Last name alphabetically**
```sql
SELECT MAX(name) AS last_name_alphabetically
FROM employees;
```

**Output:**
```
┌──────────────────────────┐
│ last_name_alphabetically │
├──────────────────────────┤
│ Sarah                    │
└──────────────────────────┘
```
*S comes after D, E, J, M alphabetically*

---

## 8. GROUP BY Clause

### Theory

**GROUP BY** divides rows into groups based on column values and applies aggregate functions to each group separately.

**Key Concepts:**

**1. Purpose:**
- Create subgroups in your data
- Apply aggregate functions to each group
- Answer questions like "total per department", "average by category"

**2. How It Works:**
- Groups rows with the same values in specified columns
- Each group gets one row in the result
- Aggregate functions calculate separately for each group

**3. Critical Rule:**
- **Every column in SELECT must be either:**
  - In the GROUP BY clause, OR
  - Inside an aggregate function
- This is called the "GROUP BY rule"

**4. Multiple Columns:**
- Can group by multiple columns
- Creates groups for unique combinations

**5. Execution Order:**
```
FROM → WHERE → GROUP BY → Aggregate Functions → HAVING → SELECT → ORDER BY
```

### Syntax
```sql
SELECT column1, AGGREGATE_FUNCTION(column2)
FROM table_name
WHERE condition
GROUP BY column1;
```

### Examples with Output

**Example 1: Count employees per department**
```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department;
```

**Output:**
```
┌────────────┬────────────────┐
│ department │ employee_count │
├────────────┼────────────────┤
│ Sales      │ 2              │
│ IT         │ 2              │
│ HR         │ 1              │
└────────────┴────────────────┘
```

---

**Example 2: Total salary per department**
```sql
SELECT department, SUM(salary) AS total_salary
FROM employees
GROUP BY department;
```

**Output:**
```
┌────────────┬──────────────┐
│ department │ total_salary │
├────────────┼──────────────┤
│ Sales      │ 95000        │
│ IT         │ 125000       │
│ HR         │ 55000        │
└────────────┴──────────────┘
```
*Sales: 50000 + 45000 = 95000*
*IT: 60000 + 65000 = 125000*

---

**Example 3: Average salary per department**
```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department;
```

**Output:**
```
┌────────────┬────────────┐
│ department │ avg_salary │
├────────────┼────────────┤
│ Sales      │ 47500.00   │
│ IT         │ 62500.00   │
│ HR         │ 55000.00   │
└────────────┴────────────┘
```

---

**Example 4: Count orders per customer**
```sql
SELECT customer_id, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id;
```

**Output:**
```
┌─────────────┬─────────────┐
│ customer_id │ order_count │
├─────────────┼─────────────┤
│ 1           │ 2           │
│ 2           │ 2           │
│ 3           │ 1           │
└─────────────┴─────────────┘
```

---

**Example 5: Total order amount per customer**
```sql
SELECT customer_id, SUM(amount) AS total_spent
FROM orders
GROUP BY customer_id;
```

**Output:**
```
┌─────────────┬─────────────┐
│ customer_id │ total_spent │
├─────────────┼─────────────┤
│ 1           │ 550         │
│ 2           │ 250         │
│ 3           │ 200         │
└─────────────┴─────────────┘
```
*Customer 1: 250 + 300 = 550*
*Customer 2: 150 + 100 = 250*

---

**Example 6: Count products per category**
```sql
SELECT category, COUNT(*) AS product_count, AVG(price) AS avg_price
FROM products
GROUP BY category;
```

**Output:**
```
┌─────────────┬───────────────┬───────────┐
│  category   │ product_count │ avg_price │
├─────────────┼───────────────┼───────────┤
│ Electronics │ 3             │ 541.67    │
│ Furniture   │ 2             │ 225.00    │
└─────────────┴───────────────┴───────────┘
```

---

**Example 7: Min and Max salary per department**
```sql
SELECT department, 
       MIN(salary) AS min_salary, 
       MAX(salary) AS max_salary
FROM employees
GROUP BY department;
```

**Output:**
```
┌────────────┬────────────┬────────────┐
│ department │ min_salary │ max_salary │
├────────────┼────────────┼────────────┤
│ Sales      │ 45000      │ 50000      │
│ IT         │ 60000      │ 65000      │
│ HR         │ 55000      │ 55000      │
└────────────┴────────────┴────────────┘
```

---

**Example 8: GROUP BY with WHERE**
```sql
SELECT department, COUNT(*) AS high_earners
FROM employees
WHERE salary > 50000
GROUP BY department;
```

**Output:**
```
┌────────────┬──────────────┐
│ department │ high_earners │
├────────────┼──────────────┤
│ IT         │ 2            │
│ HR         │ 1            │
└────────────┴──────────────┘
```
*WHERE filters before grouping: only Sarah (60000), Emma (65000), David (55000)*

---

**Example 9: Multiple aggregates**
```sql
SELECT department,
       COUNT(*) AS emp_count,
       SUM(salary) AS total_sal,
       AVG(salary) AS avg_sal,
       MIN(salary) AS min_sal,
       MAX(salary) AS max_sal
FROM employees
GROUP BY department;
```

**Output:**
```
┌────────────┬───────────┬───────────┬──────────┬─────────┬─────────┐
│ department │ emp_count │ total_sal │  avg_sal │ min_sal │ max_sal │
├────────────┼───────────┼───────────┼──────────┼─────────┼─────────┤
│ Sales      │ 2         │ 95000     │ 47500.00 │ 45000   │ 50000   │
│ IT         │ 2         │ 125000    │ 62500.00 │ 60000   │ 65000   │
│ HR         │ 1         │ 55000     │ 55000.00 │ 55000   │ 55000   │
└────────────┴───────────┴───────────┴──────────┴─────────┴─────────┘
```

---

## 9. HAVING Clause

### Theory

**HAVING** filters groups created by GROUP BY based on aggregate function results. It's like WHERE but for groups.

**Key Concepts:**

**1. Purpose:**
- Filter groups after aggregation
- Apply conditions to aggregate results
- Answer questions like "departments with more than 5 employees"

**2. When to Use:**
- When you need to filter based on aggregate values
- After GROUP BY has created groups
- With conditions on COUNT, SUM, AVG, MIN, MAX

**3. HAVING vs WHERE:**
- **WHERE:** Filters individual rows BEFORE grouping
- **HAVING:** Filters groups AFTER aggregation
- Cannot use WHERE with aggregate functions

**4. Can Include:**
- Aggregate functions (COUNT, SUM, AVG, etc.)
- Columns that appear in GROUP BY
- Both combined

**5. Execution Order:**
```
FROM → WHERE → GROUP BY → Aggregate → HAVING → SELECT → ORDER BY
```

### Syntax
```sql
SELECT column1, AGGREGATE_FUNCTION(column2)
FROM table_name
WHERE condition                    -- Filters rows before grouping
GROUP BY column1
HAVING AGGREGATE_FUNCTION(column2) condition;  -- Filters groups after aggregation
```

### Examples with Output

**Example 1: Departments with more than 1 employee**
```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 1;
```

**Output:**
```
┌────────────┬────────────────┐
│ department │ employee_count │
├────────────┼────────────────┤
│ Sales      │ 2              │
│ IT         │ 2              │
└────────────┴────────────────┘
```
*HR is excluded because it has only 1 employee*

---

**Example 2: Departments with average salary above 50000**
```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;
```

**Output:**
```
┌────────────┬────────────┐
│ department │ avg_salary │
├────────────┼────────────┤
│ IT         │ 62500.00   │
│ HR         │ 55000.00   │
└────────────┴────────────┘
```
*Sales excluded: avg salary 47500 is not > 50000*

---

**Example 3: Customers with total spending over 300**
```sql
SELECT customer_id, SUM(amount) AS total_spent
FROM orders
GROUP BY customer_id
HAVING SUM(amount) > 300;
```

**Output:**
```
┌─────────────┬─────────────┐
│ customer_id │ total_spent │
├─────────────┼─────────────┤
│ 1           │ 550         │
└─────────────┴─────────────┘
```
*Customer 2 (250) and Customer 3 (200) are excluded*

---

**Example 4: Order statuses with count >= 2**
```sql
SELECT status, COUNT(*) AS order_count
FROM orders
GROUP BY status
HAVING COUNT(*) >= 2;
```

**Output:**
```
┌─────────┬─────────────┐
│ status  │ order_count │
├─────────┼─────────────┤
│ Shipped │ 3           │
└─────────┴─────────────┘
```
*Pending (1) and Canceled (1) are excluded*

---

**Example 5: Departments with total salary over 100000**
```sql
SELECT department, SUM(salary) AS total_salary
FROM employees
GROUP BY department
HAVING SUM(salary) > 100000;
```

**Output:**
```
┌────────────┬──────────────┐
│ department │ total_salary │
├────────────┼──────────────┤
│ IT         │ 125000       │
└────────────┴──────────────┘
```
*Sales (95000) and HR (55000) are excluded*

---

**Example 6: Categories with max price > 300**
```sql
SELECT category, MAX(price) AS max_price
FROM products
GROUP BY category
HAVING MAX(price) > 300;
```

**Output:**
```
┌─────────────┬───────────┐
│  category   │ max_price │
├─────────────┼───────────┤
│ Electronics │ 1200      │
└─────────────┴───────────┘
```
*Furniture excluded: max price 300 is not > 300*

---

**Example 7: HAVING with multiple conditions**
```sql
SELECT department, 
       COUNT(*) AS emp_count,
       AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING COUNT(*) >= 2 AND AVG(salary) > 50000;
```

**Output:**
```
┌────────────┬───────────┬────────────┐
│ department │ emp_count │ avg_salary │
├────────────┼───────────┼────────────┤
│ IT         │ 2         │ 62500.00   │
└────────────┴───────────┴────────────┘
```
*Sales excluded: avg_salary 47500 not > 50000*
*HR excluded: count is 1, not >= 2*

---

## 10. WHERE vs HAVING

### Theory

Understanding the difference between WHERE and HAVING is crucial for writing correct queries.

**WHERE Clause:**
- **Filters:** Individual rows
- **When:** BEFORE grouping and aggregation
- **Can Use:** Column values, comparisons, logical operators
- **Cannot Use:** Aggregate functions
- **Purpose:** Reduce data before calculations

**HAVING Clause:**
- **Filters:** Groups (aggregated results)
- **When:** AFTER grouping and aggregation
- **Can Use:** Aggregate functions, grouped columns
- **Cannot Use:** (Generally) non-aggregated, non-grouped columns
- **Purpose:** Filter based on calculated values

**Execution Order:**
```
1. FROM     - Get the table
2. WHERE    - Filter individual rows
3. GROUP BY - Create groups
4. Aggregate- Calculate functions (COUNT, SUM, etc.)
5. HAVING   - Filter groups
6. SELECT   - Choose columns to display
7. ORDER BY - Sort results
```

**Key Principle:**
- Use **WHERE** to filter data you don't want to aggregate
- Use **HAVING** to filter aggregated results

### Comparison Table

| Aspect | WHERE | HAVING |
|--------|-------|--------|
| Filters | Individual rows | Groups |
| Applied | Before GROUP BY | After GROUP BY |
| Works with aggregates | ❌ No | ✅ Yes |
| Works with columns | ✅ Yes | ✅ Yes (if grouped) |
| Purpose | Filter source data | Filter results |
| Performance | Better (filters early) | After calculations |

### Examples with Output

**Example 1: WHERE - Filter rows before grouping**
```sql
SELECT department, COUNT(*) AS emp_count
FROM employees
WHERE salary > 50000        -- WHERE filters rows
GROUP BY department;
```

**Output:**
```
┌────────────┬───────────┐
│ department │ emp_count │
├────────────┼───────────┤
│ IT         │ 2         │
│ HR         │ 1         │
└────────────┴───────────┘
```
**Explanation:**
- WHERE filters BEFORE grouping: Only includes Sarah (60000), Emma (65000), David (55000)
- Then groups these 3 employees by department

---

**Example 2: HAVING - Filter groups after aggregation**
```sql
SELECT department, COUNT(*) AS emp_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 1;        -- HAVING filters groups
```

**Output:**
```
┌────────────┬───────────┐
│ department │ emp_count │
├────────────┼───────────┤
│ Sales      │ 2         │
│ IT         │ 2         │
└────────────┴───────────┘
```
**Explanation:**
- Groups all 5 employees first
- Then HAVING excludes groups with count <= 1

---

**Example 3: Using BOTH WHERE and HAVING**
```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
WHERE salary > 45000              -- WHERE: filters rows
GROUP BY department
HAVING AVG(salary) > 55000;       -- HAVING: filters groups
```

**Output:**
```
┌────────────┬────────────┐
│ department │ avg_salary │
├────────────┼────────────┤
│ IT         │ 62500.00   │
└────────────┴────────────┘
```
**Explanation:**
Step 1 - WHERE filters: Keep only John (50000), Sarah (60000), Emma (65000), David (55000)
Step 2 - GROUP BY: 
  - Sales: John (50000) → avg = 50000
  - IT: Sarah (60000), Emma (65000) → avg = 62500
  - HR: David (55000) → avg = 55000
Step 3 - HAVING filters: Only IT has avg > 55000

---

**Example 4: Wrong - Cannot use aggregate in WHERE**
```sql
-- ❌ WRONG - This will cause an error
SELECT department, AVG(salary) AS avg_salary
FROM employees
WHERE AVG(salary) > 50000    -- ERROR! Cannot use aggregate in WHERE
GROUP BY department;
```
**Error:** Aggregate functions are not allowed in WHERE clause

**Correct Version:**
```sql
-- ✅ CORRECT - Use HAVING for aggregates
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;
```

---

**Example 5: WHERE and HAVING together - Orders example**
```sql
SELECT customer_id, SUM(amount) AS total_amount
FROM orders
WHERE status = 'Shipped'          -- Filter: only shipped orders
GROUP BY customer_id
HAVING SUM(amount) > 250;         -- Filter: total > 250
```

**Output:**
```
┌─────────────┬──────────────┐
│ customer_id │ total_amount │
├─────────────┼──────────────┤
│ 1           │ 550          │
└─────────────┴──────────────┘
```
**Explanation:**
Step 1 - WHERE: Only orders 101, 103, 104 (status = 'Shipped')
Step 2 - GROUP BY customer:
  - Customer 1: 250 + 300 = 550
  - Customer 3: 200
Step 3 - HAVING: Only customer 1 has sum > 250

---

**Example 6: Comparison - Same question, different approaches**

**Scenario:** Find departments where employees earn more than 50000

**Approach 1: Using WHERE (filters employees first)**
```sql
SELECT department, COUNT(*) AS high_earners
FROM employees
WHERE salary > 50000
GROUP BY department;
```

**Output:**
```
┌────────────┬──────────────┐
│ department │ high_earners │
├────────────┼──────────────┤
│ IT         │ 2            │
│ HR         │ 1            │
└────────────┴──────────────┘
```

**Approach 2: Using HAVING (filters departments after averaging)**
```sql
SELECT department, COUNT(*) AS emp_count
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;
```

**Output:**
```
┌────────────┬───────────┐
│ department │ emp_count │
├────────────┼───────────┤
│ IT         │ 2         │
│ HR         │ 1         │
└────────────┴───────────┘
```

**Different Questions!**
- First query: Count employees earning > 50000 per department
- Second query: Departments where average salary > 50000

---

**Example 7: Decision Guide**

```sql
-- Question: "Show departments with high salary employees"
-- Answer: Use WHERE (filter rows by salary)
SELECT department, COUNT(*) AS high_earners
FROM employees
WHERE salary > 55000
GROUP BY department;
```

**Output:**
```
┌────────────┬──────────────┐
│ department │ high_earners │
├────────────┼──────────────┤
│ IT         │ 2            │
└────────────┴──────────────┘
```

---

```sql
-- Question: "Show departments with high average salary"
-- Answer: Use HAVING (filter groups by average)
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 55000;
```

**Output:**
```
┌────────────┬────────────┐
│ department │ avg_salary │
├────────────┼────────────┤
│ IT         │ 62500.00   │
└────────────┴────────────┘
```

---

## 11. Combining Aggregations

### Theory

You can combine multiple aggregate functions and GROUP BY clauses to create comprehensive reports.

**Common Patterns:**

**1. Multiple Aggregates:**
- Use several aggregate functions in one query
- Provides complete statistical summary

**2. Calculated Columns:**
- Combine aggregates with arithmetic
- Example: AVG(salary) * COUNT(*) for total cost

**3. GROUP BY Multiple Columns:**
- Create finer-grained groups
- Example: GROUP BY department, manager

**4. Nested Aggregations:**
- Subqueries with aggregates
- Finding values above average, etc.

### Examples with Output

**Example 1: Complete department statistics**
```sql
SELECT department,
       COUNT(*) AS employees,
       SUM(salary) AS total_sal,
       AVG(salary) AS avg_sal,
       MIN(salary) AS min_sal,
       MAX(salary) AS max_sal,
       MAX(salary) - MIN(salary) AS salary_range
FROM employees
GROUP BY department
ORDER BY total_sal DESC;
```

**Output:**
```
┌────────────┬───────────┬───────────┬──────────┬─────────┬─────────┬──────────────┐
│ department │ employees │ total_sal │  avg_sal │ min_sal │ max_sal │ salary_range │
├────────────┼───────────┼───────────┼──────────┼─────────┼─────────┼──────────────┤
│ IT         │ 2         │ 125000    │ 62500.00 │ 60000   │ 65000   │ 5000         │
│ Sales      │ 2         │ 95000     │ 47500.00 │ 45000   │ 50000   │ 5000         │
│ HR         │ 1         │ 55000     │ 55000.00 │ 55000   │ 55000   │ 0            │
└────────────┴───────────┴───────────┴──────────┴─────────┴─────────┴──────────────┘
```

---

**Example 2: GROUP BY multiple columns**
```sql
SELECT department, manager, COUNT(*) AS team_size
FROM employees
WHERE manager IS NOT NULL
GROUP BY department, manager
ORDER BY department, team_size DESC;
```

**Output:**
```
┌────────────┬─────────┬───────────┐
│ department │ manager │ team_size │
├────────────┼─────────┼───────────┤
│ IT         │ Bob     │ 2         │
│ Sales      │ Alice   │ 2         │
└────────────┴─────────┴───────────┘
```

---

**Example 3: Customer order summary**
```sql
SELECT customer_id,
       COUNT(*) AS total_orders,
       SUM(amount) AS total_spent,
       AVG(amount) AS avg_order_value,
       MIN(amount) AS smallest_order,
       MAX(amount) AS largest_order
FROM orders
GROUP BY customer_id
ORDER BY total_spent DESC;
```

**Output:**
```
┌─────────────┬──────────────┬─────────────┬─────────────────┬────────────────┬───────────────┐
│ customer_id │ total_orders │ total_spent │ avg_order_value │ smallest_order │ largest_order │
├─────────────┼──────────────┼─────────────┼─────────────────┼────────────────┼───────────────┤
│ 1           │ 2            │ 550         │ 275.00          │ 250            │ 300           │
│ 2           │ 2            │ 250         │ 125.00          │ 100            │ 150           │
│ 3           │ 1            │ 200         │ 200.00          │ 200            │ 200           │
└─────────────┴──────────────┴─────────────┴─────────────────┴────────────────┴───────────────┘
```

---

## 12. Common Patterns

### Pattern 1: Finding Totals and Counts
```sql
SELECT category,
       COUNT(*) AS product_count,
       SUM(price) AS total_value
FROM products
GROUP BY category;
```

**Output:**
```
┌─────────────┬───────────────┬─────────────┐
│  category   │ product_count │ total_value │
├─────────────┼───────────────┼─────────────┤
│ Electronics │ 3             │ 1625        │
│ Furniture   │ 2             │ 450         │
└─────────────┴───────────────┴─────────────┘
```

---

### Pattern 2: Top N Groups
```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
ORDER BY avg_salary DESC
LIMIT 2;
```

**Output:**
```
┌────────────┬────────────┐
│ department │ avg_salary │
├────────────┼────────────┤
│ IT         │ 62500.00   │
│ HR         │ 55000.00   │
└────────────┴────────────┘
```

---

### Pattern 3: Percentage Calculations
```sql
SELECT department,
       COUNT(*) AS dept_count,
       ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM employees), 2) AS percentage
FROM employees
GROUP BY department;
```

**Output:**
```
┌────────────┬────────────┬────────────┐
│ department │ dept_count │ percentage │
├────────────┼────────────┼────────────┤
│ Sales      │ 2          │ 40.00      │
│ IT         │ 2          │ 40.00      │
│ HR         │ 1          │ 20.00      │
└────────────┴────────────┴────────────┘
```

---

### Pattern 4: Conditional Aggregation
```sql
SELECT customer_id,
       COUNT(*) AS total_orders,
       SUM(CASE WHEN status = 'Shipped' THEN amount ELSE 0 END) AS shipped_total,
       SUM(CASE WHEN status = 'Pending' THEN amount ELSE 0 END) AS pending_total
FROM orders
GROUP BY customer_id;
```

**Output:**
```
┌─────────────┬──────────────┬────────────────┬───────────────┐
│ customer_id │ total_orders │ shipped_total  │ pending_total │
├─────────────┼──────────────┼────────────────┼───────────────┤
│ 1           │ 2            │ 550            │ 0             │
│ 2           │ 2            │ 0              │ 150           │
│ 3           │ 1            │ 200            │ 0             │
└─────────────┴──────────────┴────────────────┴───────────────┘
```

---

## 13. Best Practices

### DO's ✅

1. **Always include grouped columns in SELECT**
   - If you GROUP BY department, SELECT should include department

2. **Use meaningful aliases**
   ```sql
   SELECT department, COUNT(*) AS employee_count  -- Good
   ```

3. **Use WHERE to filter before aggregation**
   - Improves performance
   - Reduces data before calculations

4. **Use HAVING for aggregate conditions**
   - Correct syntax for filtering aggregated results

5. **Order results for readability**
   ```sql
   ORDER BY COUNT(*) DESC  -- Most common first
   ```

6. **Combine multiple aggregates for complete picture**
   - COUNT, SUM, AVG together provide full statistics

---

### DON'Ts ❌

1. **Don't use aggregate functions in WHERE**
   ```sql
   -- ❌ WRONG
   WHERE COUNT(*) > 5
   
   -- ✅ CORRECT
   HAVING COUNT(*) > 5
   ```

2. **Don't SELECT ungrouped, unaggregated columns**
   ```sql
   -- ❌ WRONG
   SELECT name, department, COUNT(*)  -- name is not grouped!
   GROUP BY department
   
   -- ✅ CORRECT
   SELECT department, COUNT(*)
   GROUP BY department
   ```

3. **Don't forget NULL handling**
   - Remember: COUNT(column) excludes NULLs
   - SUM, AVG ignore NULLs

4. **Don't use HAVING when WHERE would work**
   ```sql
   -- ❌ Less efficient
   GROUP BY department
   HAVING department = 'IT'
   
   -- ✅ More efficient
   WHERE department = 'IT'
   GROUP BY department
   ```

5. **Don't over-aggregate**
   - Only calculate what you need
   - Each aggregate has computation cost

---

### Common Errors and Solutions

**Error 1: Column not in GROUP BY**
```sql
-- ❌ ERROR
SELECT name, department, COUNT(*)
FROM employees
GROUP BY department;
-- Error: 'name' must be in GROUP BY or aggregate function

-- ✅ SOLUTION
SELECT department, COUNT(*)
FROM employees
GROUP BY department;
```

---

**Error 2: Aggregate in WHERE**
```sql
-- ❌ ERROR
SELECT department, AVG(salary)
FROM employees
WHERE AVG(salary) > 50000
GROUP BY department;
-- Error: Cannot use aggregate in WHERE

-- ✅ SOLUTION
SELECT department, AVG(salary)
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;
```

---

**Error 3: Forgetting GROUP BY with aggregates**
```sql
-- ❌ WRONG (returns only one row)
SELECT department, COUNT(*)
FROM employees;

-- ✅ CORRECT
SELECT department, COUNT(*)
FROM employees
GROUP BY department;
```

---

## Quick Reference

### Aggregate Functions Summary

| Function | Purpose | Ignores NULL | Return Type |
|----------|---------|--------------|-------------|
| COUNT(*) | Count all rows | No | Integer |
| COUNT(col) | Count non-NULL | Yes | Integer |
| SUM(col) | Total of values | Yes | Numeric |
| AVG(col) | Average value | Yes | Numeric/Decimal |
| MIN(col) | Smallest value | Yes | Same as column |
| MAX(col) | Largest value | Yes | Same as column |

### Clause Order

```sql
SELECT column, AGGREGATE(column)
FROM table
WHERE row_condition
GROUP BY column
HAVING aggregate_condition
ORDER BY column
LIMIT n;
```

### WHERE vs HAVING

| Use WHERE when... | Use HAVING when... |
|-------------------|-------------------|
| Filtering individual rows | Filtering groups |
| Before grouping | After aggregation |
| Not using aggregates | Using aggregates |
| Better performance | Need aggregate results |

---

## Summary

**Aggregate Functions** provide powerful data summarization:
- **COUNT**: Number of rows or values
- **SUM**: Total of numeric values
- **AVG**: Average of numeric values
- **MIN**: Smallest value
- **MAX**: Largest value

**GROUP BY** creates subgroups for aggregation:
- Groups rows with same values
- Each group gets one result row
- All SELECT columns must be grouped or aggregated

**HAVING** filters aggregated results:
- Applied after GROUP BY
- Works with aggregate functions
- Different from WHERE (which filters before grouping)

**Key Principle**: 
- Use **WHERE** to filter what you aggregate
- Use **HAVING** to filter aggregated results

Master these concepts—they're essential for data analysis, reporting, and understanding your data!

---

**End of SQL Aggregations & Grouping Guide**
