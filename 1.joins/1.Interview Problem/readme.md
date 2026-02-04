# SQL JOIN Interview Problems - Number Tables

## Table of Contents
1. [Sample Tables Setup](#sample-tables-setup)
2. [Problem 1: INNER JOIN - Find Common Numbers](#problem-1-inner-join---find-common-numbers)
3. [Problem 2: LEFT JOIN - All from Table A](#problem-2-left-join---all-from-table-a)
4. [Problem 3: RIGHT JOIN - All from Table B](#problem-3-right-join---all-from-table-b)
5. [Problem 4: FULL OUTER JOIN - All Numbers](#problem-4-full-outer-join---all-numbers)
6. [Problem 5: CROSS JOIN - All Combinations](#problem-5-cross-join---all-combinations)
7. [Problem 6: Find Numbers Only in Table A](#problem-6-find-numbers-only-in-table-a)
8. [Problem 7: Find Numbers Only in Table B](#problem-7-find-numbers-only-in-table-b)
9. [Problem 8: Find Numbers in Either Table (No Duplicates)](#problem-8-find-numbers-in-either-table-no-duplicates)
10. [Problem 9: Count Matching Numbers](#problem-9-count-matching-numbers)
11. [Problem 10: Self JOIN - Number Pairs](#problem-10-self-join---number-pairs)

---

## Sample Tables Setup

We have two simple tables with single columns containing numbers:

### Table A
```sql
CREATE TABLE TableA (
    num INT
);

INSERT INTO TableA (num) VALUES (1), (2), (3), (4), (5);
```

| num |
|-----|
| 1   |
| 2   |
| 3   |
| 4   |
| 5   |

### Table B
```sql
CREATE TABLE TableB (
    num INT
);

INSERT INTO TableB (num) VALUES (3), (4), (5), (6), (7);
```

| num |
|-----|
| 3   |
| 4   |
| 5   |
| 6   |
| 7   |

---

## Problem 1: INNER JOIN - Find Common Numbers

### ❓ Question
**Find all numbers that exist in BOTH Table A and Table B.**

### 📘 Definition Applied
**INNER JOIN retrieves only the records that have matching values in both tables based on the specified join condition.**
- It only matches the columns mentioned in the JOIN condition
- Records that do not have a match in both tables are excluded

### 💡 Solution
```sql
SELECT A.num
FROM TableA A
INNER JOIN TableB B
ON A.num = B.num;
```

### 📊 Result
| num |
|-----|
| 3   |
| 4   |
| 5   |

### 🔍 Explanation
- Only matches columns: `A.num = B.num`
- Numbers 3, 4, 5 exist in BOTH tables → included
- Numbers 1, 2 (only in A) → excluded
- Numbers 6, 7 (only in B) → excluded

### 🔢 Formula
```
INNER JOIN = A ∩ B
Result = {3, 4, 5} (intersection only)
```

---

## Problem 2: LEFT JOIN - All from Table A

### ❓ Question
**Get all numbers from Table A, and show if they exist in Table B.**

### 📘 Definition Applied
**LEFT JOIN first performs an INNER JOIN to fetch all matching records from both tables.**
- Additionally, it includes any remaining records from the left table that did not find a match
- For columns from the right table that do not have a match, NULL values will be returned
- The left table is the one mentioned before the LEFT JOIN keyword

### 💡 Solution
```sql
SELECT A.num AS num_from_A, B.num AS num_from_B
FROM TableA A
LEFT JOIN TableB B
ON A.num = B.num;
```

### 📊 Result
| num_from_A | num_from_B |
|------------|------------|
| 1          | NULL       |
| 2          | NULL       |
| 3          | 3          |
| 4          | 4          |
| 5          | 5          |

### 🔍 Explanation
**Step 1:** Perform INNER JOIN
- Matches: 3, 4, 5

**Step 2:** Add remaining left table records
- Numbers 1, 2 from Table A (no match in B) → Added with NULL

### 🔢 Formula
```
LEFT JOIN = INNER JOIN + Unmatched Left Records
Result = All 5 numbers from A + NULL for unmatched B columns
```

---

## Problem 3: RIGHT JOIN - All from Table B

### ❓ Question
**Get all numbers from Table B, and show if they exist in Table A.**

### 📘 Definition Applied
**RIGHT JOIN first performs an INNER JOIN to get all matching records.**
- It then includes any remaining records from the right table that were not part of the initial inner join
- The right table is the one mentioned after the RIGHT JOIN keyword, and it is given priority

### 💡 Solution
```sql
SELECT A.num AS num_from_A, B.num AS num_from_B
FROM TableA A
RIGHT JOIN TableB B
ON A.num = B.num;
```

### 📊 Result
| num_from_A | num_from_B |
|------------|------------|
| 3          | 3          |
| 4          | 4          |
| 5          | 5          |
| NULL       | 6          |
| NULL       | 7          |

### 🔍 Explanation
**Step 1:** Perform INNER JOIN
- Matches: 3, 4, 5

**Step 2:** Add remaining right table records
- Numbers 6, 7 from Table B (no match in A) → Added with NULL

### 🔢 Formula
```
RIGHT JOIN = INNER JOIN + Unmatched Right Records
Result = All 5 numbers from B + NULL for unmatched A columns
```

---

## Problem 4: FULL OUTER JOIN - All Numbers

### ❓ Question
**Get ALL numbers from both tables, showing matches and non-matches.**

### 📘 Definition Applied
**FULL OUTER JOIN combines the results of both LEFT JOIN and RIGHT JOIN.**
- Returns all matching records between two tables
- Plus all unmatched records from the left table
- Plus all unmatched records from the right table

### 💡 Solution
```sql
SELECT A.num AS num_from_A, B.num AS num_from_B
FROM TableA A
FULL OUTER JOIN TableB B
ON A.num = B.num;
```

### 📊 Result
| num_from_A | num_from_B |
|------------|------------|
| 1          | NULL       |
| 2          | NULL       |
| 3          | 3          |
| 4          | 4          |
| 5          | 5          |
| NULL       | 6          |
| NULL       | 7          |

### 🔍 Explanation
**Step 1:** Perform INNER JOIN
- Matches: 3, 4, 5

**Step 2:** Add unmatched from left table
- Numbers 1, 2 from A

**Step 3:** Add unmatched from right table
- Numbers 6, 7 from B

### 🔢 Formula
```
FULL OUTER JOIN = LEFT JOIN + RIGHT JOIN
Result = Everything from both tables (7 total rows)
```

### 💻 Alternative for MySQL (No FULL OUTER JOIN support)
```sql
SELECT A.num AS num_from_A, B.num AS num_from_B
FROM TableA A
LEFT JOIN TableB B ON A.num = B.num
UNION
SELECT A.num AS num_from_A, B.num AS num_from_B
FROM TableA A
RIGHT JOIN TableB B ON A.num = B.num;
```

---

## Problem 5: CROSS JOIN - All Combinations

### ❓ Question
**Create all possible pairs of numbers from Table A and Table B.**

### 📘 Definition Applied
**CROSS JOIN returns the Cartesian product of the two tables.**
- Every row from the first table is combined with every row from the second table
- It does not require a join condition

### 💡 Solution
```sql
SELECT A.num AS num_from_A, B.num AS num_from_B
FROM TableA A
CROSS JOIN TableB B;
```

### 📊 Result (25 rows total)
| num_from_A | num_from_B |
|------------|------------|
| 1          | 3          |
| 1          | 4          |
| 1          | 5          |
| 1          | 6          |
| 1          | 7          |
| 2          | 3          |
| 2          | 4          |
| 2          | 5          |
| 2          | 6          |
| 2          | 7          |
| 3          | 3          |
| 3          | 4          |
| 3          | 5          |
| 3          | 6          |
| 3          | 7          |
| 4          | 3          |
| 4          | 4          |
| 4          | 5          |
| 4          | 6          |
| 4          | 7          |
| 5          | 3          |
| 5          | 4          |
| 5          | 5          |
| 5          | 6          |
| 5          | 7          |

### 🔍 Explanation
- Table A has 5 rows
- Table B has 5 rows
- Result = 5 × 5 = 25 rows (every combination)
- No join condition needed

### 🔢 Formula
```
CROSS JOIN = A × B
Result = 5 × 5 = 25 rows (all combinations)
```

---

## Problem 6: Find Numbers Only in Table A

### ❓ Question
**Find numbers that exist in Table A but NOT in Table B.**

### 📘 Definition Applied
**LEFT JOIN first performs an INNER JOIN, then includes remaining records from the left table.**
- We use WHERE clause to filter only the unmatched records (where B.num IS NULL)

### 💡 Solution
```sql
SELECT A.num
FROM TableA A
LEFT JOIN TableB B
ON A.num = B.num
WHERE B.num IS NULL;
```

### 📊 Result
| num |
|-----|
| 1   |
| 2   |

### 🔍 Explanation
**Step 1:** LEFT JOIN gives us all from A:
```
A.num | B.num
1     | NULL
2     | NULL
3     | 3
4     | 4
5     | 5
```

**Step 2:** Filter WHERE B.num IS NULL
- Only rows where B.num is NULL remain
- These are numbers that exist only in A

### 🔢 Formula
```
Numbers only in A = LEFT JOIN result WHERE right side IS NULL
Result = {1, 2}
```

---

## Problem 7: Find Numbers Only in Table B

### ❓ Question
**Find numbers that exist in Table B but NOT in Table A.**

### 📘 Definition Applied
**RIGHT JOIN first performs an INNER JOIN, then includes remaining records from the right table.**
- We use WHERE clause to filter only the unmatched records (where A.num IS NULL)

### 💡 Solution
```sql
SELECT B.num
FROM TableA A
RIGHT JOIN TableB B
ON A.num = B.num
WHERE A.num IS NULL;
```

### 📊 Result
| num |
|-----|
| 6   |
| 7   |

### 🔍 Explanation
**Step 1:** RIGHT JOIN gives us all from B:
```
A.num | B.num
3     | 3
4     | 4
5     | 5
NULL  | 6
NULL  | 7
```

**Step 2:** Filter WHERE A.num IS NULL
- Only rows where A.num is NULL remain
- These are numbers that exist only in B

### 🔢 Formula
```
Numbers only in B = RIGHT JOIN result WHERE left side IS NULL
Result = {6, 7}
```

### 💻 Alternative using LEFT JOIN
```sql
SELECT B.num
FROM TableB B
LEFT JOIN TableA A
ON B.num = A.num
WHERE A.num IS NULL;
```

---

## Problem 8: Find Numbers in Either Table (No Duplicates)

### ❓ Question
**Find all unique numbers that exist in Table A OR Table B (or both).**

### 📘 Definition Applied
**FULL OUTER JOIN combines LEFT JOIN and RIGHT JOIN.**
- We use COALESCE to get the non-NULL value from either table

### 💡 Solution
```sql
SELECT COALESCE(A.num, B.num) AS num
FROM TableA A
FULL OUTER JOIN TableB B
ON A.num = B.num;
```

### 📊 Result
| num |
|-----|
| 1   |
| 2   |
| 3   |
| 4   |
| 5   |
| 6   |
| 7   |

### 🔍 Explanation
FULL OUTER JOIN gives us:
```
A.num | B.num | COALESCE(A.num, B.num)
1     | NULL  | 1
2     | NULL  | 2
3     | 3     | 3
4     | 4     | 4
5     | 5     | 5
NULL  | 6     | 6
NULL  | 7     | 7
```

COALESCE returns the first non-NULL value, giving us all unique numbers.

### 🔢 Formula
```
All unique numbers = FULL OUTER JOIN with COALESCE
Result = {1, 2, 3, 4, 5, 6, 7} (union of both tables)
```

### 💻 Alternative using UNION
```sql
SELECT num FROM TableA
UNION
SELECT num FROM TableB;
```

---

## Problem 9: Count Matching Numbers

### ❓ Question
**Count how many numbers are common between Table A and Table B.**

### 📘 Definition Applied
**INNER JOIN retrieves only the records that have matching values in both tables.**
- We use COUNT to count the matching records

### 💡 Solution
```sql
SELECT COUNT(*) AS matching_count
FROM TableA A
INNER JOIN TableB B
ON A.num = B.num;
```

### 📊 Result
| matching_count |
|----------------|
| 3              |

### 🔍 Explanation
- INNER JOIN returns only matching records: 3, 4, 5
- COUNT(*) counts these matching rows
- Result = 3 common numbers

### 🔢 Formula
```
Matching count = COUNT(INNER JOIN result)
Result = 3 (numbers 3, 4, 5)
```

---

## Problem 10: Self JOIN - Number Pairs

### ❓ Question
**Find all pairs of numbers from Table A where the first number is less than the second number.**

### 📘 Definition Applied
**SELF JOIN is a regular JOIN where a table is joined with itself.**
- It requires aliases for the table to distinguish between the two instances
- Used to match records within the same table

### 💡 Solution
```sql
SELECT A1.num AS first_num, A2.num AS second_num
FROM TableA A1
INNER JOIN TableA A2
ON A1.num < A2.num;
```

### 📊 Result
| first_num | second_num |
|-----------|------------|
| 1         | 2          |
| 1         | 3          |
| 1         | 4          |
| 1         | 5          |
| 2         | 3          |
| 2         | 4          |
| 2         | 5          |
| 3         | 4          |
| 3         | 5          |
| 4         | 5          |

### 🔍 Explanation
- Table A is used twice with aliases A1 and A2
- A1 represents the first number
- A2 represents the second number
- Condition: A1.num < A2.num ensures first < second
- This creates all valid pairs where first number is smaller

### 🔢 Formula
```
SELF JOIN = Table A JOIN Table A (with aliases)
Result = All pairs where A1.num < A2.num
Total pairs = n×(n-1)/2 = 5×4/2 = 10 pairs
```

---

## 🎯 Summary Table

| Problem | JOIN Type | Result Count | What It Returns |
|---------|-----------|--------------|-----------------|
| 1       | INNER     | 3            | Common numbers (3,4,5) |
| 2       | LEFT      | 5            | All from A with B info |
| 3       | RIGHT     | 5            | All from B with A info |
| 4       | FULL OUTER| 7            | All numbers from both |
| 5       | CROSS     | 25           | All combinations |
| 6       | LEFT + WHERE | 2         | Only in A (1,2) |
| 7       | RIGHT + WHERE | 2        | Only in B (6,7) |
| 8       | FULL OUTER | 7           | Unique numbers |
| 9       | INNER + COUNT | 1         | Count of matches (3) |
| 10      | SELF      | 10           | Number pairs |

---

## 🔑 Key Takeaways

### INNER JOIN
- Only matching records from both tables
- Matches specified columns only
- Excludes non-matching records

### LEFT JOIN
- INNER JOIN first (matches)
- Then adds unmatched left records with NULL
- Left table = before LEFT JOIN keyword

### RIGHT JOIN
- INNER JOIN first (matches)
- Then adds unmatched right records with NULL
- Right table = after RIGHT JOIN keyword (priority)

### FULL OUTER JOIN
- Combines LEFT JOIN + RIGHT JOIN
- All matching + all unmatched from both tables
- Everything from both tables

### CROSS JOIN
- Cartesian product
- Every row from first × every row from second
- No join condition required

### SELF JOIN
- Table joined with itself
- Requires aliases
- Used for comparing rows within same table

---

## 💡 Interview Tips

1. **Understand the definition**: Know exactly what each JOIN does step by step
2. **Draw Venn diagrams**: Visualize which records are included
3. **Count rows**: Always know the expected result count
4. **NULL handling**: Remember WHERE IS NULL filters unmatched records
5. **Self JOIN**: Always use different aliases for the same table
6. **CROSS JOIN**: Watch out for large result sets (M × N)
7. **Practice**: Draw tables and trace through the JOIN steps manually

---

## 🚀 Practice Exercise

Try these variations with the same tables:

1. Find numbers that exist in ONLY one table (not both)
2. Find the maximum number from matched records
3. Find pairs where the sum equals 8
4. Count how many numbers are unique to each table
5. Find numbers in A that are greater than all numbers in B

Happy Learning! 🎓
