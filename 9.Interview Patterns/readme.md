# Real SQL Interview Patterns - Must Practice Guide

## Index
1. [Introduction](#introduction)
2. [Second Highest Salary](#second-highest-salary)
   - [Method 1: LIMIT with OFFSET](#method-1-limit-with-offset)
   - [Method 2: Subquery with MAX](#method-2-subquery-with-max)
   - [Method 3: DENSE_RANK](#method-3-dense_rank)
   - [Method 4: NTH_VALUE](#method-4-nth_value)
3. [Duplicate Records](#duplicate-records)
   - [Finding Duplicates](#finding-duplicates)
   - [Removing Duplicates](#removing-duplicates)
   - [Keeping One Copy](#keeping-one-copy)
4. [Top N per Department](#top-n-per-department)
   - [Using ROW_NUMBER](#using-row_number)
   - [Using RANK](#using-rank)
   - [Using Correlated Subquery](#using-correlated-subquery)
5. [Gaps in Dates](#gaps-in-dates)
   - [Finding Missing Dates](#finding-missing-dates)
   - [Finding Date Ranges](#finding-date-ranges)
   - [Identifying Gaps](#identifying-gaps)
6. [Consecutive Records](#consecutive-records)
   - [Finding Consecutive Sequences](#finding-consecutive-sequences)
   - [Login Streaks](#login-streaks)
   - [Grouping Consecutive Rows](#grouping-consecutive-rows)
7. [Running Totals](#running-totals)
   - [Using Window Functions](#using-window-functions)
   - [Using Self Join](#using-self-join)
   - [Cumulative Aggregates](#cumulative-aggregates)
8. [Slowly Changing Dimensions](#slowly-changing-dimensions)
   - [Type 1: Overwrite](#type-1-overwrite)
   - [Type 2: Track History](#type-2-track-history)
   - [Type 3: Limited History](#type-3-limited-history)
9. [Pivot / Unpivot](#pivot--unpivot)
   - [Pivot: Rows to Columns](#pivot-rows-to-columns)
   - [Unpivot: Columns to Rows](#unpivot-columns-to-rows)
   - [Dynamic Pivot](#dynamic-pivot)
10. [Complex Business Logic with CASE](#complex-business-logic-with-case)
    - [Multi-condition Logic](#multi-condition-logic)
    - [Nested CASE Statements](#nested-case-statements)
    - [CASE in Aggregations](#case-in-aggregations)
11. [Interview Tips](#interview-tips)
12. [Practice Problems](#practice-problems)

---

## Introduction

These are the **most common SQL interview patterns** asked by top tech companies (Google, Amazon, Meta, Microsoft, etc.).

**Why These Patterns:**
- Test practical SQL knowledge
- Measure problem-solving ability
- Assess optimization thinking
- Reveal real-world experience

**How to Practice:**
1. Understand the problem
2. Try solving without looking
3. Learn multiple approaches
4. Understand trade-offs
5. Practice variations

---

## Second Highest Salary

**Problem:** Find the second highest salary from employees table.

**Sample Data - employees table:**

| employee_id | name | salary |
|-------------|------|--------|
| 1 | Alice | 90000 |
| 2 | Bob | 80000 |
| 3 | Carol | 85000 |
| 4 | David | 90000 |
| 5 | Eve | 70000 |

**Expected Output:**

| second_highest_salary |
|-----------------------|
| 85000 |

---

### Method 1: LIMIT with OFFSET

**Approach:** Order by salary DESC, skip first, take second.

```sql
SELECT DISTINCT salary AS second_highest_salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
```

**Pros:** Simple and readable  
**Cons:** Returns NULL if less than 2 distinct salaries

**Step-by-step:**
1. Order salaries: 90000, 90000, 85000, 80000, 70000
2. DISTINCT: 90000, 85000, 80000, 70000
3. OFFSET 1: Skip 90000
4. LIMIT 1: Take 85000

---

### Method 2: Subquery with MAX

**Approach:** Find MAX salary that's less than the overall MAX.

```sql
SELECT MAX(salary) AS second_highest_salary
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

**Pros:** Handles edge cases (returns NULL if no second highest)  
**Cons:** Two scans of the table

**Step-by-step:**
1. Inner query: MAX(salary) = 90000
2. Outer query: MAX(salary) WHERE salary < 90000 = 85000

---

### Method 3: DENSE_RANK

**Approach:** Rank salaries, filter for rank 2.

```sql
SELECT salary AS second_highest_salary
FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rank
    FROM employees
) ranked
WHERE rank = 2;
```

**Pros:** Easy to modify for Nth highest (change rank = 2 to rank = N)  
**Cons:** More complex for just second highest

**Intermediate Result:**

| salary | rank |
|--------|------|
| 90000 | 1 |
| 90000 | 1 |
| 85000 | 2 |
| 80000 | 3 |
| 70000 | 4 |

---

### Method 4: NTH_VALUE

**Approach:** Use NTH_VALUE window function.

```sql
SELECT DISTINCT NTH_VALUE(salary, 2) OVER (
    ORDER BY salary DESC
    RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
) AS second_highest_salary
FROM employees;
```

**Pros:** Direct solution for Nth value  
**Cons:** Requires frame clause, database-specific

---

### Variation: Nth Highest Salary

**Problem:** Find the Nth highest salary (generic solution).

```sql
-- N = 3 for third highest
WITH ranked_salaries AS (
    SELECT DISTINCT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rank
    FROM employees
)
SELECT salary
FROM ranked_salaries
WHERE rank = 3;  -- Change to desired N
```

**For N = 3, Output:**

| salary |
|--------|
| 80000 |

---

## Duplicate Records

### Finding Duplicates

**Problem:** Find all duplicate email addresses.

**Sample Data - users table:**

| user_id | email | name |
|---------|-------|------|
| 1 | alice@example.com | Alice |
| 2 | bob@example.com | Bob |
| 3 | alice@example.com | Alice Smith |
| 4 | carol@example.com | Carol |
| 5 | bob@example.com | Bob Jones |

**Solution 1: GROUP BY with HAVING**

```sql
SELECT email, COUNT(*) AS duplicate_count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

**Output:**

| email | duplicate_count |
|-------|-----------------|
| alice@example.com | 2 |
| bob@example.com | 2 |

**Solution 2: Window Function**

```sql
SELECT user_id, email, name
FROM (
    SELECT *, COUNT(*) OVER (PARTITION BY email) AS email_count
    FROM users
) subquery
WHERE email_count > 1
ORDER BY email;
```

**Output:**

| user_id | email | name |
|---------|-------|------|
| 1 | alice@example.com | Alice |
| 3 | alice@example.com | Alice Smith |
| 2 | bob@example.com | Bob |
| 5 | bob@example.com | Bob Jones |

---

### Removing Duplicates

**Problem:** Delete duplicate records, keep only one.

**Solution 1: Using ROW_NUMBER (PostgreSQL, SQL Server)**

```sql
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

**Before:**
| user_id | email |
|---------|-------|
| 1 | alice@example.com |
| 3 | alice@example.com |

**After:**
| user_id | email |
|---------|-------|
| 1 | alice@example.com |

---

### Keeping One Copy

**Problem:** Get unique records (keep lowest user_id).

```sql
SELECT email, MIN(user_id) AS user_id, MIN(name) AS name
FROM users
GROUP BY email;
```

**Output:**

| email | user_id | name |
|-------|---------|------|
| alice@example.com | 1 | Alice |
| bob@example.com | 2 | Bob |
| carol@example.com | 4 | Carol |

---

## Top N per Department

**Problem:** Find top 3 highest-paid employees in each department.

**Sample Data - employees table:**

| employee_id | name | department | salary |
|-------------|------|------------|--------|
| 1 | Alice | Sales | 90000 |
| 2 | Bob | Sales | 80000 |
| 3 | Carol | Sales | 85000 |
| 4 | David | IT | 95000 |
| 5 | Eve | IT | 88000 |
| 6 | Frank | IT | 92000 |
| 7 | Grace | Sales | 75000 |

---

### Using ROW_NUMBER

**Best for:** Exactly N results per group, ties broken by some order.

```sql
SELECT employee_id, name, department, salary
FROM (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rn
    FROM employees
) ranked
WHERE rn <= 3;
```

**Output:**

| employee_id | name | department | salary |
|-------------|------|------------|--------|
| 4 | David | IT | 95000 |
| 6 | Frank | IT | 92000 |
| 5 | Eve | IT | 88000 |
| 1 | Alice | Sales | 90000 |
| 3 | Carol | Sales | 85000 |
| 2 | Bob | Sales | 80000 |

---

### Using RANK

**Best for:** Include all ties (may return more than N per group).

```sql
SELECT employee_id, name, department, salary
FROM (
    SELECT *,
           RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
    FROM employees
) ranked
WHERE rank <= 3;
```

**If two people tied for 3rd:** Both included.

---

### Using Correlated Subquery

**Alternative approach** (less efficient but works everywhere).

```sql
SELECT e1.employee_id, e1.name, e1.department, e1.salary
FROM employees e1
WHERE (
    SELECT COUNT(DISTINCT e2.salary)
    FROM employees e2
    WHERE e2.department = e1.department
      AND e2.salary >= e1.salary
) <= 3
ORDER BY e1.department, e1.salary DESC;
```

**How it works:** Counts how many salaries in same dept are >= current salary. If count ≤ 3, include.

---

## Gaps in Dates

**Problem:** Find missing dates in a sequence.

**Sample Data - attendance table:**

| employee_id | attendance_date |
|-------------|-----------------|
| 101 | 2024-01-01 |
| 101 | 2024-01-02 |
| 101 | 2024-01-04 |
| 101 | 2024-01-05 |

**Missing:** 2024-01-03

---

### Finding Missing Dates

**Solution: Generate series and find gaps**

```sql
-- PostgreSQL
WITH date_range AS (
    SELECT generate_series(
        (SELECT MIN(attendance_date) FROM attendance),
        (SELECT MAX(attendance_date) FROM attendance),
        '1 day'::interval
    )::date AS expected_date
)
SELECT dr.expected_date AS missing_date
FROM date_range dr
LEFT JOIN attendance a ON dr.expected_date = a.attendance_date
WHERE a.attendance_date IS NULL;
```

**Output:**

| missing_date |
|--------------|
| 2024-01-03 |

**MySQL Version:**

```sql
-- Requires a numbers table or recursive CTE
WITH RECURSIVE dates AS (
    SELECT MIN(attendance_date) AS date FROM attendance
    UNION ALL
    SELECT DATE_ADD(date, INTERVAL 1 DAY)
    FROM dates
    WHERE date < (SELECT MAX(attendance_date) FROM attendance)
)
SELECT dates.date AS missing_date
FROM dates
LEFT JOIN attendance ON dates.date = attendance.attendance_date
WHERE attendance.attendance_date IS NULL;
```

---

### Finding Date Ranges

**Problem:** Find continuous date ranges.

```sql
WITH date_groups AS (
    SELECT 
        attendance_date,
        attendance_date - ROW_NUMBER() OVER (ORDER BY attendance_date) * INTERVAL '1 day' AS group_id
    FROM attendance
)
SELECT 
    MIN(attendance_date) AS range_start,
    MAX(attendance_date) AS range_end,
    COUNT(*) AS days_count
FROM date_groups
GROUP BY group_id
ORDER BY range_start;
```

**Output:**

| range_start | range_end | days_count |
|-------------|-----------|------------|
| 2024-01-01 | 2024-01-02 | 2 |
| 2024-01-04 | 2024-01-05 | 2 |

---

### Identifying Gaps

**Problem:** Find gaps between consecutive records.

```sql
SELECT 
    current_date,
    next_date,
    next_date - current_date - 1 AS gap_days
FROM (
    SELECT 
        attendance_date AS current_date,
        LEAD(attendance_date) OVER (ORDER BY attendance_date) AS next_date
    FROM attendance
) gaps
WHERE next_date - current_date > 1;
```

**Output:**

| current_date | next_date | gap_days |
|--------------|-----------|----------|
| 2024-01-02 | 2024-01-04 | 1 |

---

## Consecutive Records

**Problem:** Find users who logged in on consecutive days.

**Sample Data - logins table:**

| user_id | login_date |
|---------|------------|
| 1 | 2024-01-01 |
| 1 | 2024-01-02 |
| 1 | 2024-01-03 |
| 1 | 2024-01-05 |
| 2 | 2024-01-01 |
| 2 | 2024-01-03 |

---

### Finding Consecutive Sequences

**Solution:** Group consecutive dates using difference technique.

```sql
WITH grouped_logins AS (
    SELECT 
        user_id,
        login_date,
        login_date - ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) * INTERVAL '1 day' AS group_id
    FROM (SELECT DISTINCT user_id, login_date FROM logins) unique_logins
)
SELECT 
    user_id,
    MIN(login_date) AS streak_start,
    MAX(login_date) AS streak_end,
    COUNT(*) AS consecutive_days
FROM grouped_logins
GROUP BY user_id, group_id
HAVING COUNT(*) >= 3  -- At least 3 consecutive days
ORDER BY user_id, streak_start;
```

**Output:**

| user_id | streak_start | streak_end | consecutive_days |
|---------|--------------|------------|------------------|
| 1 | 2024-01-01 | 2024-01-03 | 3 |

**Explanation:**
- Row numbers: 1, 2, 3 for consecutive dates
- Subtracting ROW_NUMBER from date creates same value for consecutive dates
- Group by this value to find streaks

---

### Login Streaks

**Problem:** Find users with login streaks of 7+ days.

```sql
WITH login_streaks AS (
    SELECT 
        user_id,
        login_date,
        login_date - ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) * INTERVAL '1 day' AS streak_group
    FROM (SELECT DISTINCT user_id, login_date FROM logins) unique_logins
),
streak_counts AS (
    SELECT 
        user_id,
        MIN(login_date) AS streak_start,
        MAX(login_date) AS streak_end,
        COUNT(*) AS streak_length
    FROM login_streaks
    GROUP BY user_id, streak_group
)
SELECT user_id, streak_start, streak_end, streak_length
FROM streak_counts
WHERE streak_length >= 7
ORDER BY streak_length DESC;
```

---

### Grouping Consecutive Rows

**Problem:** Group consecutive status changes.

**Sample Data - device_status table:**

| device_id | timestamp | status |
|-----------|-----------|--------|
| 1 | 10:00 | online |
| 1 | 10:05 | online |
| 1 | 10:10 | offline |
| 1 | 10:15 | offline |
| 1 | 10:20 | online |

```sql
WITH status_changes AS (
    SELECT 
        device_id,
        timestamp,
        status,
        LAG(status) OVER (PARTITION BY device_id ORDER BY timestamp) AS prev_status
    FROM device_status
),
grouped_status AS (
    SELECT 
        *,
        SUM(CASE WHEN status != prev_status OR prev_status IS NULL THEN 1 ELSE 0 END) 
            OVER (PARTITION BY device_id ORDER BY timestamp) AS status_group
    FROM status_changes
)
SELECT 
    device_id,
    status,
    MIN(timestamp) AS period_start,
    MAX(timestamp) AS period_end
FROM grouped_status
GROUP BY device_id, status, status_group
ORDER BY device_id, period_start;
```

**Output:**

| device_id | status | period_start | period_end |
|-----------|--------|--------------|------------|
| 1 | online | 10:00 | 10:05 |
| 1 | offline | 10:10 | 10:15 |
| 1 | online | 10:20 | 10:20 |

---

## Running Totals

**Problem:** Calculate cumulative sum of sales by date.

**Sample Data - sales table:**

| sale_date | amount |
|-----------|--------|
| 2024-01-01 | 100 |
| 2024-01-02 | 150 |
| 2024-01-03 | 200 |
| 2024-01-04 | 120 |

---

### Using Window Functions

**Modern approach (SQL:2003+)**

```sql
SELECT 
    sale_date,
    amount,
    SUM(amount) OVER (ORDER BY sale_date) AS running_total,
    AVG(amount) OVER (ORDER BY sale_date) AS running_avg
FROM sales
ORDER BY sale_date;
```

**Output:**

| sale_date | amount | running_total | running_avg |
|-----------|--------|---------------|-------------|
| 2024-01-01 | 100 | 100 | 100.00 |
| 2024-01-02 | 150 | 250 | 125.00 |
| 2024-01-03 | 200 | 450 | 150.00 |
| 2024-01-04 | 120 | 570 | 142.50 |

---

### Using Self Join

**Legacy approach (works in old databases)**

```sql
SELECT 
    s1.sale_date,
    s1.amount,
    SUM(s2.amount) AS running_total
FROM sales s1
JOIN sales s2 ON s2.sale_date <= s1.sale_date
GROUP BY s1.sale_date, s1.amount
ORDER BY s1.sale_date;
```

**Output:** Same as above, but slower (O(n²) vs O(n log n))

---

### Cumulative Aggregates

**Problem:** Running total by category.

```sql
SELECT 
    sale_date,
    category,
    amount,
    SUM(amount) OVER (PARTITION BY category ORDER BY sale_date) AS category_running_total,
    SUM(amount) OVER (ORDER BY sale_date) AS overall_running_total
FROM sales
ORDER BY sale_date, category;
```

**Sample Data:**

| sale_date | category | amount |
|-----------|----------|--------|
| 2024-01-01 | Electronics | 100 |
| 2024-01-01 | Clothing | 50 |
| 2024-01-02 | Electronics | 150 |

**Output:**

| sale_date | category | amount | category_running_total | overall_running_total |
|-----------|----------|--------|------------------------|----------------------|
| 2024-01-01 | Clothing | 50 | 50 | 50 |
| 2024-01-01 | Electronics | 100 | 100 | 150 |
| 2024-01-02 | Electronics | 150 | 250 | 300 |

---

## Slowly Changing Dimensions

**Scenario:** Track customer address changes over time.

**Sample Data - customers table:**

| customer_id | name | address | city |
|-------------|------|---------|------|
| 1 | Alice | 123 Main St | NYC |
| 2 | Bob | 456 Oak Ave | LA |

**Alice moves to Boston.**

---

### Type 1: Overwrite

**Strategy:** Simply update the record (no history).

```sql
UPDATE customers
SET address = '789 Elm St', city = 'Boston'
WHERE customer_id = 1;
```

**Result:**

| customer_id | name | address | city |
|-------------|------|---------|------|
| 1 | Alice | 789 Elm St | Boston |
| 2 | Bob | 456 Oak Ave | LA |

**Pros:** Simple, saves space  
**Cons:** No history, can't recreate past reports

---

### Type 2: Track History

**Strategy:** Create new record for each change, mark current.

**Table Structure:**

```sql
CREATE TABLE customers_scd2 (
    surrogate_key SERIAL PRIMARY KEY,
    customer_id INT,
    name VARCHAR(100),
    address VARCHAR(200),
    city VARCHAR(50),
    effective_date DATE,
    expiry_date DATE,
    is_current BOOLEAN
);
```

**Initial Data:**

| surrogate_key | customer_id | name | address | city | effective_date | expiry_date | is_current |
|---------------|-------------|------|---------|------|----------------|-------------|------------|
| 1 | 1 | Alice | 123 Main St | NYC | 2023-01-01 | 9999-12-31 | TRUE |
| 2 | 2 | Bob | 456 Oak Ave | LA | 2023-01-01 | 9999-12-31 | TRUE |

**After Alice moves (2024-02-01):**

```sql
-- Step 1: Close old record
UPDATE customers_scd2
SET expiry_date = '2024-01-31', is_current = FALSE
WHERE customer_id = 1 AND is_current = TRUE;

-- Step 2: Insert new record
INSERT INTO customers_scd2 (customer_id, name, address, city, effective_date, expiry_date, is_current)
VALUES (1, 'Alice', '789 Elm St', 'Boston', '2024-02-01', '9999-12-31', TRUE);
```

**Result:**

| surrogate_key | customer_id | name | address | city | effective_date | expiry_date | is_current |
|---------------|-------------|------|---------|------|----------------|-------------|------------|
| 1 | 1 | Alice | 123 Main St | NYC | 2023-01-01 | 2024-01-31 | FALSE |
| 2 | 2 | Bob | 456 Oak Ave | LA | 2023-01-01 | 9999-12-31 | TRUE |
| 3 | 1 | Alice | 789 Elm St | Boston | 2024-02-01 | 9999-12-31 | TRUE |

**Query current state:**
```sql
SELECT * FROM customers_scd2 WHERE is_current = TRUE;
```

**Query historical state (as of 2024-01-15):**
```sql
SELECT * FROM customers_scd2
WHERE '2024-01-15' BETWEEN effective_date AND expiry_date;
```

**Pros:** Complete history, point-in-time queries  
**Cons:** More storage, complex queries

---

### Type 3: Limited History

**Strategy:** Keep previous value in separate column.

**Table Structure:**

```sql
CREATE TABLE customers_scd3 (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    current_address VARCHAR(200),
    current_city VARCHAR(50),
    previous_address VARCHAR(200),
    previous_city VARCHAR(50),
    effective_date DATE
);
```

**After Alice moves:**

| customer_id | name | current_address | current_city | previous_address | previous_city | effective_date |
|-------------|------|-----------------|--------------|------------------|---------------|----------------|
| 1 | Alice | 789 Elm St | Boston | 123 Main St | NYC | 2024-02-01 |
| 2 | Bob | 456 Oak Ave | LA | NULL | NULL | 2023-01-01 |

**Pros:** Simple, one previous value  
**Cons:** Limited history (only 1 previous)

---

## Pivot / Unpivot

### Pivot: Rows to Columns

**Problem:** Convert monthly sales data from rows to columns.

**Sample Data - monthly_sales table:**

| product | month | sales |
|---------|-------|-------|
| Laptop | Jan | 1000 |
| Laptop | Feb | 1200 |
| Laptop | Mar | 1100 |
| Phone | Jan | 800 |
| Phone | Feb | 900 |
| Phone | Mar | 850 |

**Desired Output:**

| product | Jan | Feb | Mar |
|---------|-----|-----|-----|
| Laptop | 1000 | 1200 | 1100 |
| Phone | 800 | 900 | 850 |

---

**Solution 1: CASE with GROUP BY**

```sql
SELECT 
    product,
    SUM(CASE WHEN month = 'Jan' THEN sales ELSE 0 END) AS Jan,
    SUM(CASE WHEN month = 'Feb' THEN sales ELSE 0 END) AS Feb,
    SUM(CASE WHEN month = 'Mar' THEN sales ELSE 0 END) AS Mar
FROM monthly_sales
GROUP BY product;
```

**Solution 2: PIVOT (SQL Server)**

```sql
SELECT *
FROM monthly_sales
PIVOT (
    SUM(sales)
    FOR month IN ([Jan], [Feb], [Mar])
) AS pivot_table;
```

**Solution 3: Conditional Aggregation (PostgreSQL)**

```sql
SELECT 
    product,
    MAX(CASE WHEN month = 'Jan' THEN sales END) AS Jan,
    MAX(CASE WHEN month = 'Feb' THEN sales END) AS Feb,
    MAX(CASE WHEN month = 'Mar' THEN sales END) AS Mar
FROM monthly_sales
GROUP BY product;
```

---

### Unpivot: Columns to Rows

**Problem:** Convert quarterly sales from columns to rows.

**Sample Data - quarterly_sales table:**

| product | Q1 | Q2 | Q3 | Q4 |
|---------|----|----|----|----|
| Laptop | 3000 | 3200 | 3100 | 3400 |
| Phone | 2400 | 2500 | 2600 | 2700 |

**Desired Output:**

| product | quarter | sales |
|---------|---------|-------|
| Laptop | Q1 | 3000 |
| Laptop | Q2 | 3200 |
| Laptop | Q3 | 3100 |
| Laptop | Q4 | 3400 |
| Phone | Q1 | 2400 |
| ... | ... | ... |

---

**Solution 1: UNION ALL**

```sql
SELECT product, 'Q1' AS quarter, Q1 AS sales FROM quarterly_sales
UNION ALL
SELECT product, 'Q2' AS quarter, Q2 AS sales FROM quarterly_sales
UNION ALL
SELECT product, 'Q3' AS quarter, Q3 AS sales FROM quarterly_sales
UNION ALL
SELECT product, 'Q4' AS quarter, Q4 AS sales FROM quarterly_sales
ORDER BY product, quarter;
```

**Solution 2: UNPIVOT (SQL Server)**

```sql
SELECT product, quarter, sales
FROM quarterly_sales
UNPIVOT (
    sales FOR quarter IN (Q1, Q2, Q3, Q4)
) AS unpivot_table;
```

**Solution 3: CROSS JOIN with VALUES (PostgreSQL)**

```sql
SELECT 
    q.product,
    quarters.quarter,
    CASE quarters.quarter
        WHEN 'Q1' THEN q.Q1
        WHEN 'Q2' THEN q.Q2
        WHEN 'Q3' THEN q.Q3
        WHEN 'Q4' THEN q.Q4
    END AS sales
FROM quarterly_sales q
CROSS JOIN (VALUES ('Q1'), ('Q2'), ('Q3'), ('Q4')) AS quarters(quarter)
ORDER BY q.product, quarters.quarter;
```

---

### Dynamic Pivot

**Problem:** Pivot with unknown number of columns.

```sql
-- PostgreSQL: Using crosstab
CREATE EXTENSION IF NOT EXISTS tablefunc;

SELECT *
FROM crosstab(
    'SELECT product, month, sales FROM monthly_sales ORDER BY 1,2'
) AS ct(product VARCHAR, Jan INT, Feb INT, Mar INT);
```

---

## Complex Business Logic with CASE

### Multi-condition Logic

**Problem:** Categorize products based on multiple conditions.

**Sample Data - products table:**

| product_id | price | stock | category |
|------------|-------|-------|----------|
| 1 | 50 | 100 | Electronics |
| 2 | 500 | 5 | Electronics |
| 3 | 20 | 200 | Clothing |
| 4 | 200 | 0 | Electronics |

**Solution:**

```sql
SELECT 
    product_id,
    price,
    stock,
    category,
    CASE 
        WHEN stock = 0 THEN 'Out of Stock'
        WHEN stock < 10 AND price > 100 THEN 'Low Stock - High Value'
        WHEN stock < 10 THEN 'Low Stock'
        WHEN stock > 100 THEN 'Overstocked'
        ELSE 'Normal'
    END AS inventory_status,
    CASE
        WHEN price < 50 THEN 'Budget'
        WHEN price BETWEEN 50 AND 200 THEN 'Mid-Range'
        ELSE 'Premium'
    END AS price_tier
FROM products;
```

**Output:**

| product_id | price | stock | category | inventory_status | price_tier |
|------------|-------|-------|----------|------------------|------------|
| 1 | 50 | 100 | Electronics | Overstocked | Mid-Range |
| 2 | 500 | 5 | Electronics | Low Stock - High Value | Premium |
| 3 | 20 | 200 | Clothing | Overstocked | Budget |
| 4 | 200 | 0 | Electronics | Out of Stock | Mid-Range |

---

### Nested CASE Statements

**Problem:** Calculate commission based on sales tier and department.

```sql
SELECT 
    employee_id,
    sales_amount,
    department,
    CASE department
        WHEN 'Sales' THEN
            CASE 
                WHEN sales_amount > 100000 THEN sales_amount * 0.15
                WHEN sales_amount > 50000 THEN sales_amount * 0.10
                ELSE sales_amount * 0.05
            END
        WHEN 'Support' THEN
            CASE 
                WHEN sales_amount > 50000 THEN sales_amount * 0.08
                ELSE sales_amount * 0.03
            END
        ELSE sales_amount * 0.02
    END AS commission
FROM employee_sales;
```

**Sample Data:**

| employee_id | sales_amount | department |
|-------------|--------------|------------|
| 1 | 120000 | Sales |
| 2 | 60000 | Support |

**Output:**

| employee_id | sales_amount | department | commission |
|-------------|--------------|------------|------------|
| 1 | 120000 | Sales | 18000 |
| 2 | 60000 | Support | 4800 |

---

### CASE in Aggregations

**Problem:** Count different types of events.

```sql
SELECT 
    user_id,
    COUNT(*) AS total_events,
    SUM(CASE WHEN event_type = 'login' THEN 1 ELSE 0 END) AS login_count,
    SUM(CASE WHEN event_type = 'purchase' THEN 1 ELSE 0 END) AS purchase_count,
    SUM(CASE WHEN event_type = 'logout' THEN 1 ELSE 0 END) AS logout_count,
    AVG(CASE WHEN event_type = 'purchase' THEN amount ELSE NULL END) AS avg_purchase_amount
FROM user_events
GROUP BY user_id;
```

**Sample Data - user_events:**

| user_id | event_type | amount |
|---------|------------|--------|
| 1 | login | NULL |
| 1 | purchase | 50 |
| 1 | purchase | 100 |
| 1 | logout | NULL |

**Output:**

| user_id | total_events | login_count | purchase_count | logout_count | avg_purchase_amount |
|---------|--------------|-------------|----------------|--------------|---------------------|
| 1 | 4 | 1 | 2 | 1 | 75.00 |

---

## Interview Tips

### General Strategy
1. **Clarify requirements** - Ask about edge cases, NULL handling
2. **Start simple** - Get basic query working first
3. **Optimize later** - Don't over-optimize prematurely
4. **Explain your thinking** - Talk through your approach
5. **Test edge cases** - Empty results, duplicates, NULLs

### Common Mistakes to Avoid
- Forgetting DISTINCT with ranking functions
- Not handling NULL values
- Using correlated subqueries when window functions work
- Forgetting to order by in window functions
- Not considering performance with large datasets

### Quick Decision Guide

**Need Nth highest?**
→ Use DENSE_RANK() or LIMIT/OFFSET

**Finding duplicates?**
→ Use GROUP BY with HAVING or window functions

**Top N per group?**
→ Use ROW_NUMBER() with PARTITION BY

**Missing values in sequence?**
→ Generate series and LEFT JOIN

**Consecutive records?**
→ Use ROW_NUMBER() subtraction technique

**Running totals?**
→ Use SUM() OVER (ORDER BY ...)

**Tracking history?**
→ Use SCD Type 2 with effective dates

**Rows to columns?**
→ Use CASE with GROUP BY (PIVOT)

**Complex conditions?**
→ Use CASE statements

---

## Practice Problems

### Problem 1: Employee Hierarchy

**Find employees and their managers (including level in hierarchy).**

```sql
WITH RECURSIVE employee_hierarchy AS (
    -- Anchor: Top-level employees (no manager)
    SELECT employee_id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive: Add subordinates
    SELECT e.employee_id, e.name, e.manager_id, eh.level + 1
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT * FROM employee_hierarchy ORDER BY level, employee_id;
```

---

### Problem 2: Customer Retention

**Find customers who made purchases in consecutive months.**

```sql
WITH monthly_purchases AS (
    SELECT DISTINCT 
        customer_id,
        DATE_TRUNC('month', purchase_date) AS purchase_month
    FROM purchases
),
with_next_month AS (
    SELECT 
        customer_id,
        purchase_month,
        LEAD(purchase_month) OVER (PARTITION BY customer_id ORDER BY purchase_month) AS next_month
    FROM monthly_purchases
)
SELECT DISTINCT customer_id
FROM with_next_month
WHERE next_month = purchase_month + INTERVAL '1 month';
```

---

### Problem 3: Median Salary by Department

**Calculate median salary for each department.**

```sql
SELECT 
    department,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) AS median_salary
FROM employees
GROUP BY department;

-- Alternative without PERCENTILE_CONT
SELECT department, AVG(salary) AS median_salary
FROM (
    SELECT 
        department,
        salary,
        ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary) AS rn_asc,
        ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rn_desc
    FROM employees
) ranked
WHERE ABS(rn_asc - rn_desc) <= 1
GROUP BY department;
```

---

### Problem 4: Year-over-Year Growth

**Calculate YoY sales growth for each product.**

```sql
WITH yearly_sales AS (
    SELECT 
        product_id,
        EXTRACT(YEAR FROM sale_date) AS year,
        SUM(amount) AS total_sales
    FROM sales
    GROUP BY product_id, EXTRACT(YEAR FROM sale_date)
)
SELECT 
    product_id,
    year,
    total_sales,
    LAG(total_sales) OVER (PARTITION BY product_id ORDER BY year) AS prev_year_sales,
    total_sales - LAG(total_sales) OVER (PARTITION BY product_id ORDER BY year) AS growth,
    ROUND(
        (total_sales - LAG(total_sales) OVER (PARTITION BY product_id ORDER BY year)) * 100.0 / 
        LAG(total_sales) OVER (PARTITION BY product_id ORDER BY year),
        2
    ) AS growth_pct
FROM yearly_sales;
```

---

### Problem 5: Active Users (Complex Logic)

**Define active user as: logged in at least once in last 7 days AND made purchase in last 30 days.**

```sql
WITH user_activity AS (
    SELECT 
        u.user_id,
        u.username,
        MAX(CASE WHEN l.login_date >= CURRENT_DATE - 7 THEN 1 ELSE 0 END) AS logged_in_7d,
        MAX(CASE WHEN p.purchase_date >= CURRENT_DATE - 30 THEN 1 ELSE 0 END) AS purchased_30d
    FROM users u
    LEFT JOIN logins l ON u.user_id = l.user_id
    LEFT JOIN purchases p ON u.user_id = p.user_id
    GROUP BY u.user_id, u.username
)
SELECT 
    user_id,
    username,
    CASE 
        WHEN logged_in_7d = 1 AND purchased_30d = 1 THEN 'Active'
        WHEN logged_in_7d = 1 THEN 'Engaged'
        WHEN purchased_30d = 1 THEN 'Buyer'
        ELSE 'Inactive'
    END AS user_status
FROM user_activity;
```

---

## Summary

### Pattern Quick Reference

| Pattern | Key Technique | Complexity |
|---------|--------------|------------|
| **Nth Highest** | DENSE_RANK() or LIMIT OFFSET | Medium |
| **Duplicates** | GROUP BY HAVING or ROW_NUMBER() | Easy |
| **Top N per Group** | ROW_NUMBER() with PARTITION BY | Medium |
| **Missing Dates** | Generate series + LEFT JOIN | Hard |
| **Consecutive Records** | ROW_NUMBER() subtraction | Hard |
| **Running Totals** | SUM() OVER (ORDER BY) | Medium |
| **SCD** | Effective dates + is_current flag | Hard |
| **Pivot** | CASE with GROUP BY | Medium |
| **Complex Logic** | Nested CASE statements | Medium |

### Interview Success Formula

1. **Understand the problem** (5 min)
2. **Plan your approach** (3 min)
3. **Write the query** (10 min)
4. **Test and refine** (5 min)
5. **Explain complexity** (2 min)

### Must-Know Techniques

✅ Window functions (ROW_NUMBER, RANK, LEAD, LAG, SUM OVER)  
✅ CTEs for complex queries  
✅ Self-joins for comparisons  
✅ CASE statements for business logic  
✅ GROUP BY with HAVING for aggregations  
✅ Date arithmetic for gaps/sequences  
✅ Subquery optimization  

**Practice these patterns and you'll ace any SQL interview!** 🎯
