# Window Functions - Complete Guide (Interview Gold)

## Index
1. [Introduction](#introduction)
2. [Window Function Basics](#window-function-basics)
3. [Ranking Functions](#ranking-functions)
   - [ROW_NUMBER](#row_number)
   - [RANK](#rank)
   - [DENSE_RANK](#dense_rank)
   - [Comparison: ROW_NUMBER vs RANK vs DENSE_RANK](#comparison-row_number-vs-rank-vs-dense_rank)
4. [Offset Functions](#offset-functions)
   - [LEAD](#lead)
   - [LAG](#lag)
5. [Aggregate Window Functions](#aggregate-window-functions)
   - [SUM() OVER()](#sum-over)
   - [AVG() OVER()](#avg-over)
   - [COUNT() OVER()](#count-over)
6. [Window Function Components](#window-function-components)
   - [PARTITION BY](#partition-by)
   - [ORDER BY in Window](#order-by-in-window)
   - [Frame Clauses (ROWS BETWEEN)](#frame-clauses-rows-between)
7. [Interview Use Cases](#interview-use-cases)
   - [Top N per Group](#top-n-per-group)
   - [Running Totals](#running-totals)
   - [Previous/Next Row Comparison](#previousnext-row-comparison)
   - [Deduplication](#deduplication)
8. [Best Practices](#best-practices)

---

## Introduction

**Window Functions** perform calculations across a set of rows related to the current row, **WITHOUT collapsing rows** like GROUP BY does.

**Key Difference from GROUP BY:**
- **GROUP BY**: Collapses rows into groups → Returns fewer rows
- **Window Functions**: Keep all rows → Add calculated columns

**Syntax:**
```sql
function_name() OVER (
    [PARTITION BY column]
    [ORDER BY column]
    [ROWS BETWEEN ... AND ...]
)
```

---

## Window Function Basics

**Components:**
1. **Window Function**: The calculation (ROW_NUMBER, SUM, etc.)
2. **OVER Clause**: Defines the window (set of rows)
3. **PARTITION BY**: Divides rows into groups (optional)
4. **ORDER BY**: Orders rows within the window (optional)
5. **Frame Clause**: Defines which rows to include (optional)

**Mental Model:**
Think of a window function as looking through a "window" at related rows while staying on the current row.

---

## Ranking Functions

Ranking functions assign a rank to each row within a partition.

### ROW_NUMBER

Assigns a **unique sequential number** to each row, starting from 1.

**Theory:**
- Always returns unique numbers (1, 2, 3, 4...)
- Even if values are tied, each gets a different number
- Order matters (based on ORDER BY clause)

**Example:**

```sql
SELECT 
    name,
    department,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num
FROM employees;
```

**Sample Data - employees table:**

| name | department | salary |
|------|------------|--------|
| Alice | Sales | 70000 |
| Bob | IT | 70000 |
| Carol | Sales | 60000 |
| David | IT | 55000 |

**Output:**

| name | department | salary | row_num |
|------|------------|--------|---------|
| Alice | Sales | 70000 | 1 |
| Bob | IT | 70000 | 2 |
| Carol | Sales | 60000 | 3 |
| David | IT | 55000 | 4 |

*Explanation:* Alice and Bob have same salary, but ROW_NUMBER assigns them different numbers (1, 2) based on arbitrary order.

---

### RANK

Assigns ranks with **gaps** when there are ties.

**Theory:**
- Tied values get the same rank
- Next rank skips numbers (1, 2, 2, 4, 5...)
- Gap size = number of tied rows

**Example:**

```sql
SELECT 
    name,
    department,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS rank
FROM employees;
```

**Sample Data - employees table:**

| name | department | salary |
|------|------------|--------|
| Alice | Sales | 70000 |
| Bob | IT | 70000 |
| Carol | Sales | 60000 |
| David | IT | 60000 |
| Eve | HR | 55000 |

**Output:**

| name | department | salary | rank |
|------|------------|--------|------|
| Alice | Sales | 70000 | 1 |
| Bob | IT | 70000 | 1 |
| Carol | Sales | 60000 | 3 |
| David | IT | 60000 | 3 |
| Eve | HR | 55000 | 5 |

*Explanation:* Alice and Bob both get rank 1. Next rank is 3 (skipping 2). Carol and David get rank 3, next is 5.

---

### DENSE_RANK

Assigns ranks **without gaps** when there are ties.

**Theory:**
- Tied values get the same rank
- Next rank is consecutive (1, 2, 2, 3, 4...)
- No gaps in ranking sequence

**Example:**

```sql
SELECT 
    name,
    department,
    salary,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;
```

**Sample Data - employees table:**

| name | department | salary |
|------|------------|--------|
| Alice | Sales | 70000 |
| Bob | IT | 70000 |
| Carol | Sales | 60000 |
| David | IT | 60000 |
| Eve | HR | 55000 |

**Output:**

| name | department | salary | dense_rank |
|------|------------|--------|------------|
| Alice | Sales | 70000 | 1 |
| Bob | IT | 70000 | 1 |
| Carol | Sales | 60000 | 2 |
| David | IT | 60000 | 2 |
| Eve | HR | 55000 | 3 |

*Explanation:* Alice and Bob both get rank 1. Next rank is 2 (no gap). Carol and David get rank 2, next is 3.

---

### Comparison: ROW_NUMBER vs RANK vs DENSE_RANK

**Sample Data:**

| name | score |
|------|-------|
| Alice | 95 |
| Bob | 90 |
| Carol | 90 |
| David | 85 |

**Query:**

```sql
SELECT 
    name,
    score,
    ROW_NUMBER() OVER (ORDER BY score DESC) AS row_num,
    RANK() OVER (ORDER BY score DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank
FROM students;
```

**Output:**

| name | score | row_num | rank | dense_rank |
|------|-------|---------|------|------------|
| Alice | 95 | 1 | 1 | 1 |
| Bob | 90 | 2 | 2 | 2 |
| Carol | 90 | 3 | 2 | 2 |
| David | 85 | 4 | 4 | 3 |

**When to use:**
- **ROW_NUMBER**: Need unique numbering (pagination, deduplication)
- **RANK**: Olympic-style ranking (1st, 2nd, 2nd, 4th)
- **DENSE_RANK**: Consecutive ranking needed (1st, 2nd, 2nd, 3rd)

---

## Offset Functions

Offset functions access data from other rows relative to the current row.

### LEAD

Accesses data from the **next row** (following row).

**Theory:**
- Returns value from N rows ahead (default N=1)
- Returns NULL if no next row exists
- Syntax: `LEAD(column, offset, default_value) OVER (ORDER BY ...)`

**Example:**

```sql
SELECT 
    name,
    salary,
    LEAD(salary, 1) OVER (ORDER BY hire_date) AS next_salary
FROM employees;
```

**Sample Data - employees table:**

| name | salary | hire_date |
|------|--------|-----------|
| Alice | 60000 | 2023-01-15 |
| Bob | 70000 | 2023-02-20 |
| Carol | 65000 | 2023-03-10 |

**Output:**

| name | salary | next_salary |
|------|--------|-------------|
| Alice | 60000 | 70000 |
| Bob | 70000 | 65000 |
| Carol | 65000 | NULL |

*Explanation:* Alice's next hire is Bob (70000). Bob's next is Carol (65000). Carol has no next hire (NULL).

---

### LAG

Accesses data from the **previous row** (preceding row).

**Theory:**
- Returns value from N rows behind (default N=1)
- Returns NULL if no previous row exists
- Syntax: `LAG(column, offset, default_value) OVER (ORDER BY ...)`

**Example:**

```sql
SELECT 
    name,
    salary,
    LAG(salary, 1, 0) OVER (ORDER BY hire_date) AS prev_salary,
    salary - LAG(salary, 1, 0) OVER (ORDER BY hire_date) AS salary_diff
FROM employees;
```

**Sample Data - employees table:**

| name | salary | hire_date |
|------|--------|-----------|
| Alice | 60000 | 2023-01-15 |
| Bob | 70000 | 2023-02-20 |
| Carol | 65000 | 2023-03-10 |

**Output:**

| name | salary | prev_salary | salary_diff |
|------|--------|-------------|-------------|
| Alice | 60000 | 0 | 60000 |
| Bob | 70000 | 60000 | 10000 |
| Carol | 65000 | 70000 | -5000 |

*Explanation:* Alice has no previous hire (default 0). Bob's previous is Alice. Carol's previous is Bob. Difference calculated accordingly.

---

## Aggregate Window Functions

Aggregate functions with OVER clause keep all rows while performing calculations.

### SUM() OVER()

Calculates sum across a window of rows.

**Theory:**
- Without PARTITION BY: Cumulative sum across all rows
- With PARTITION BY: Sum resets for each partition
- With ORDER BY: Running/cumulative sum

**Example - Running Total:**

```sql
SELECT 
    order_date,
    amount,
    SUM(amount) OVER (ORDER BY order_date) AS running_total
FROM orders;
```

**Sample Data - orders table:**

| order_date | amount |
|------------|--------|
| 2024-01-01 | 100 |
| 2024-01-02 | 150 |
| 2024-01-03 | 200 |
| 2024-01-04 | 120 |

**Output:**

| order_date | amount | running_total |
|------------|--------|---------------|
| 2024-01-01 | 100 | 100 |
| 2024-01-02 | 150 | 250 |
| 2024-01-03 | 200 | 450 |
| 2024-01-04 | 120 | 570 |

*Explanation:* Running total accumulates: 100, 100+150=250, 250+200=450, 450+120=570.

---

### AVG() OVER()

Calculates average across a window of rows.

**Theory:**
- Can calculate overall average or moving average
- With ORDER BY and frame clause: Moving average

**Example - Moving Average (3-day):**

```sql
SELECT 
    date,
    sales,
    AVG(sales) OVER (
        ORDER BY date 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_3day
FROM daily_sales;
```

**Sample Data - daily_sales table:**

| date | sales |
|------|-------|
| 2024-01-01 | 100 |
| 2024-01-02 | 150 |
| 2024-01-03 | 200 |
| 2024-01-04 | 180 |

**Output:**

| date | sales | moving_avg_3day |
|------|-------|-----------------|
| 2024-01-01 | 100 | 100.00 |
| 2024-01-02 | 150 | 125.00 |
| 2024-01-03 | 200 | 150.00 |
| 2024-01-04 | 180 | 176.67 |

*Explanation:* 
- Day 1: avg(100) = 100
- Day 2: avg(100, 150) = 125
- Day 3: avg(100, 150, 200) = 150
- Day 4: avg(150, 200, 180) = 176.67

---

### COUNT() OVER()

Counts rows across a window.

**Example:**

```sql
SELECT 
    department,
    name,
    COUNT(*) OVER (PARTITION BY department) AS dept_count
FROM employees;
```

**Sample Data:**

| name | department |
|------|------------|
| Alice | Sales |
| Bob | Sales |
| Carol | IT |

**Output:**

| department | name | dept_count |
|------------|------|------------|
| Sales | Alice | 2 |
| Sales | Bob | 2 |
| IT | Carol | 1 |

---

## Window Function Components

### PARTITION BY

Divides the result set into partitions (groups). Window function applies separately to each partition.

**Theory:**
- Like GROUP BY but doesn't collapse rows
- Each partition is processed independently
- Resets calculations for each partition

**Example:**

```sql
SELECT 
    name,
    department,
    salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank
FROM employees;
```

**Sample Data - employees table:**

| name | department | salary |
|------|------------|--------|
| Alice | Sales | 70000 |
| Bob | Sales | 60000 |
| Carol | IT | 75000 |
| David | IT | 65000 |

**Output:**

| name | department | salary | dept_rank |
|------|------------|--------|-----------|
| Carol | IT | 75000 | 1 |
| David | IT | 65000 | 2 |
| Alice | Sales | 70000 | 1 |
| Bob | Sales | 60000 | 2 |

*Explanation:* Ranking is done separately within each department. Carol ranks 1st in IT, Alice ranks 1st in Sales.

---

### ORDER BY in Window

Defines the order of rows within each partition for the window function.

**Theory:**
- Determines which row is "next" or "previous"
- Affects running totals and cumulative calculations
- Different from outer ORDER BY (which orders final result)

**Example:**

```sql
SELECT 
    product,
    sale_date,
    quantity,
    SUM(quantity) OVER (
        PARTITION BY product 
        ORDER BY sale_date
    ) AS cumulative_qty
FROM sales;
```

**Sample Data - sales table:**

| product | sale_date | quantity |
|---------|-----------|----------|
| Widget | 2024-01-01 | 10 |
| Widget | 2024-01-02 | 15 |
| Gadget | 2024-01-01 | 20 |
| Gadget | 2024-01-03 | 25 |

**Output:**

| product | sale_date | quantity | cumulative_qty |
|---------|-----------|----------|----------------|
| Gadget | 2024-01-01 | 20 | 20 |
| Gadget | 2024-01-03 | 25 | 45 |
| Widget | 2024-01-01 | 10 | 10 |
| Widget | 2024-01-02 | 15 | 25 |

*Explanation:* Cumulative sum calculated separately for each product, ordered by sale_date.

---

### Frame Clauses (ROWS BETWEEN)

Defines the **exact set of rows** to include in the window for the current row.

**Theory:**
- Specifies a range of rows relative to current row
- Syntax: `ROWS BETWEEN <start> AND <end>`
- Common boundaries:
  - `UNBOUNDED PRECEDING`: Start of partition
  - `N PRECEDING`: N rows before current
  - `CURRENT ROW`: Current row
  - `N FOLLOWING`: N rows after current
  - `UNBOUNDED FOLLOWING`: End of partition

**Default Frame:**
- With ORDER BY: `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`
- Without ORDER BY: `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`

**Example - 7-day Moving Average:**

```sql
SELECT 
    date,
    revenue,
    AVG(revenue) OVER (
        ORDER BY date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_7day
FROM daily_revenue;
```

**Sample Data - daily_revenue table:**

| date | revenue |
|------|---------|
| 2024-01-01 | 100 |
| 2024-01-02 | 120 |
| 2024-01-03 | 110 |

**Output:**

| date | revenue | moving_avg_7day |
|------|---------|-----------------|
| 2024-01-01 | 100 | 100.00 |
| 2024-01-02 | 120 | 110.00 |
| 2024-01-03 | 110 | 110.00 |

*Explanation:* Average of current row plus up to 6 preceding rows (or fewer if not available).

**Example - Centered Moving Average:**

```sql
SELECT 
    date,
    value,
    AVG(value) OVER (
        ORDER BY date
        ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
    ) AS centered_avg
FROM measurements;
```

**Sample Data:**

| date | value |
|------|-------|
| 2024-01-01 | 10 |
| 2024-01-02 | 20 |
| 2024-01-03 | 30 |

**Output:**

| date | value | centered_avg |
|------|-------|--------------|
| 2024-01-01 | 10 | 15.00 |
| 2024-01-02 | 20 | 20.00 |
| 2024-01-03 | 30 | 25.00 |

*Explanation:* 
- Day 1: avg(10, 20) = 15 (no preceding)
- Day 2: avg(10, 20, 30) = 20
- Day 3: avg(20, 30) = 25 (no following)

---

## Interview Use Cases

### Top N per Group

**Problem:** Find top 3 highest-paid employees in each department.

**Solution:**

```sql
WITH ranked_employees AS (
    SELECT 
        name,
        department,
        salary,
        ROW_NUMBER() OVER (
            PARTITION BY department 
            ORDER BY salary DESC
        ) AS rank
    FROM employees
)
SELECT name, department, salary
FROM ranked_employees
WHERE rank <= 3;
```

**Sample Data - employees table:**

| name | department | salary |
|------|------------|--------|
| Alice | Sales | 90000 |
| Bob | Sales | 80000 |
| Carol | Sales | 70000 |
| David | Sales | 60000 |
| Eve | IT | 95000 |
| Frank | IT | 85000 |

**Output:**

| name | department | salary |
|------|------------|--------|
| Eve | IT | 95000 |
| Frank | IT | 85000 |
| Alice | Sales | 90000 |
| Bob | Sales | 80000 |
| Carol | Sales | 70000 |

*Explanation:* ROW_NUMBER ranks employees within each department. Filter keeps only top 3.

---

### Running Totals

**Problem:** Calculate cumulative sales by month.

**Solution:**

```sql
SELECT 
    month,
    sales,
    SUM(sales) OVER (ORDER BY month) AS running_total,
    SUM(sales) OVER (ORDER BY month) * 100.0 / 
        SUM(sales) OVER () AS pct_of_total
FROM monthly_sales;
```

**Sample Data - monthly_sales table:**

| month | sales |
|-------|-------|
| 2024-01 | 10000 |
| 2024-02 | 15000 |
| 2024-03 | 12000 |

**Output:**

| month | sales | running_total | pct_of_total |
|-------|-------|---------------|--------------|
| 2024-01 | 10000 | 10000 | 27.03 |
| 2024-02 | 15000 | 25000 | 67.57 |
| 2024-03 | 12000 | 37000 | 100.00 |

*Explanation:* Running total accumulates sales. Percentage shows cumulative contribution to total.

---

### Previous/Next Row Comparison

**Problem:** Calculate month-over-month growth rate.

**Solution:**

```sql
SELECT 
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) AS prev_month_revenue,
    revenue - LAG(revenue) OVER (ORDER BY month) AS revenue_change,
    ROUND(
        (revenue - LAG(revenue) OVER (ORDER BY month)) * 100.0 / 
        LAG(revenue) OVER (ORDER BY month), 
        2
    ) AS growth_rate_pct
FROM monthly_revenue;
```

**Sample Data - monthly_revenue table:**

| month | revenue |
|-------|---------|
| 2024-01 | 100000 |
| 2024-02 | 120000 |
| 2024-03 | 108000 |

**Output:**

| month | revenue | prev_month_revenue | revenue_change | growth_rate_pct |
|-------|---------|--------------------|--------------------|-----------------|
| 2024-01 | 100000 | NULL | NULL | NULL |
| 2024-02 | 120000 | 100000 | 20000 | 20.00 |
| 2024-03 | 108000 | 120000 | -12000 | -10.00 |

*Explanation:* LAG gets previous month's revenue. Calculate absolute and percentage change.

---

### Deduplication

**Problem:** Remove duplicate records, keeping only the most recent one.

**Solution:**

```sql
WITH numbered_records AS (
    SELECT 
        id,
        email,
        created_at,
        ROW_NUMBER() OVER (
            PARTITION BY email 
            ORDER BY created_at DESC
        ) AS row_num
    FROM user_registrations
)
SELECT id, email, created_at
FROM numbered_records
WHERE row_num = 1;
```

**Sample Data - user_registrations table:**

| id | email | created_at |
|----|-------|------------|
| 1 | alice@example.com | 2024-01-01 |
| 2 | bob@example.com | 2024-01-02 |
| 3 | alice@example.com | 2024-01-05 |
| 4 | alice@example.com | 2024-01-03 |

**Output:**

| id | email | created_at |
|----|-------|------------|
| 2 | bob@example.com | 2024-01-02 |
| 3 | alice@example.com | 2024-01-05 |

*Explanation:* ROW_NUMBER partitioned by email, ordered by date DESC. Row 1 is most recent for each email.

---

## Best Practices

### Performance
1. **Index ORDER BY columns** in window functions for better performance
2. **Minimize window function calls** - calculate once, reuse in outer query
3. **Use appropriate ranking function**:
   - ROW_NUMBER for unique numbering
   - RANK for gaps in ranking
   - DENSE_RANK for consecutive ranking

### Readability
4. **Use CTEs** with window functions for complex queries
5. **Name columns clearly** (running_total, prev_value, etc.)
6. **Add comments** for complex frame clauses

### Common Patterns
7. **Top N per group**: Use ROW_NUMBER/RANK with PARTITION BY
8. **Running totals**: Use SUM() OVER (ORDER BY ...)
9. **Comparisons**: Use LAG/LEAD for previous/next row
10. **Moving averages**: Use AVG() with ROWS BETWEEN frame

### Pitfalls to Avoid
11. **Don't use window functions in WHERE** - Use CTE or subquery instead
12. **Order matters** - PARTITION BY before ORDER BY in OVER clause
13. **NULL handling** - Use COALESCE with LAG/LEAD for default values
14. **Frame defaults** - Be explicit with ROWS BETWEEN when needed

---

## Summary Table

| Function | Purpose | Returns | Use Case |
|----------|---------|---------|----------|
| **ROW_NUMBER** | Unique sequential number | 1,2,3,4,5 | Pagination, deduplication |
| **RANK** | Ranking with gaps | 1,2,2,4,5 | Competition ranking |
| **DENSE_RANK** | Ranking without gaps | 1,2,2,3,4 | Grade rankings |
| **LEAD** | Next row value | Future value | Forecasting, comparisons |
| **LAG** | Previous row value | Past value | Growth rates, trends |
| **SUM() OVER** | Cumulative/windowed sum | Running total | Running totals, YTD |
| **AVG() OVER** | Cumulative/windowed average | Moving average | Smoothing, trends |

---

## Key Takeaways

1. **Window functions keep all rows** - Unlike GROUP BY which collapses rows
2. **PARTITION BY divides data** - Each partition processed independently
3. **ORDER BY defines sequence** - Critical for LAG, LEAD, running totals
4. **Frame clauses control range** - ROWS BETWEEN specifies exact rows
5. **Perfect for analytics** - Running totals, rankings, comparisons, trends

**Master these patterns and you'll excel in SQL interviews!** 🎯
