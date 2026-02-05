# Database Design Basics - Complete Guide

## Index
1. [Introduction](#introduction)
2. [Primary Key](#primary-key)
   - [What is a Primary Key](#what-is-a-primary-key)
   - [Single Column Primary Key](#single-column-primary-key)
   - [Composite Primary Key](#composite-primary-key)
   - [Natural vs Surrogate Keys](#natural-vs-surrogate-keys)
3. [Foreign Key](#foreign-key)
   - [What is a Foreign Key](#what-is-a-foreign-key)
   - [Creating Foreign Keys](#creating-foreign-keys)
   - [Referential Integrity](#referential-integrity)
   - [Cascade Operations](#cascade-operations)
4. [Constraints](#constraints)
   - [UNIQUE Constraint](#unique-constraint)
   - [CHECK Constraint](#check-constraint)
   - [NOT NULL Constraint](#not-null-constraint)
   - [DEFAULT Constraint](#default-constraint)
5. [Normalization](#normalization)
   - [What is Normalization](#what-is-normalization)
   - [First Normal Form (1NF)](#first-normal-form-1nf)
   - [Second Normal Form (2NF)](#second-normal-form-2nf)
   - [Third Normal Form (3NF)](#third-normal-form-3nf)
   - [Denormalization](#denormalization)
6. [Relationships](#relationships)
   - [One-to-One (1:1)](#one-to-one-11)
   - [One-to-Many (1:N)](#one-to-many-1n)
   - [Many-to-Many (M:N)](#many-to-many-mn)
7. [Best Practices](#best-practices)
8. [Common Interview Questions](#common-interview-questions)

---

## Introduction

**Database Design** is the process of structuring data to minimize redundancy, ensure data integrity, and optimize performance.

**Key Goals:**
- **Data Integrity**: Ensure accuracy and consistency
- **Efficiency**: Optimize storage and retrieval
- **Scalability**: Support growth
- **Maintainability**: Easy to update and modify

**Core Concepts:**
- Keys (Primary, Foreign)
- Constraints (rules for data)
- Normalization (organizing data)
- Relationships (connecting tables)

---

## Primary Key

### What is a Primary Key

A **Primary Key** is a column (or set of columns) that uniquely identifies each row in a table.

**Characteristics:**
- **Unique**: No two rows can have the same primary key value
- **Not NULL**: Cannot contain NULL values
- **Immutable**: Should not change over time
- **One per table**: Only one primary key per table

**Purpose:**
- Uniquely identify records
- Reference point for foreign keys
- Enable efficient data retrieval

---

### Single Column Primary Key

Most common type - one column serves as the primary key.

**Example:**

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);
```

**Sample Data:**

| employee_id | name | email |
|-------------|------|-------|
| 1 | Alice | alice@example.com |
| 2 | Bob | bob@example.com |
| 3 | Carol | carol@example.com |

**Key Points:**
- `employee_id` uniquely identifies each employee
- Cannot insert duplicate `employee_id`
- Cannot insert NULL `employee_id`

**Invalid Operations:**

```sql
-- ERROR: Duplicate primary key
INSERT INTO employees VALUES (1, 'David', 'david@example.com');

-- ERROR: NULL primary key
INSERT INTO employees VALUES (NULL, 'Eve', 'eve@example.com');
```

---

### Composite Primary Key

Multiple columns together form the primary key.

**Example:**

```sql
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    enrollment_date DATE,
    PRIMARY KEY (student_id, course_id)
);
```

**Sample Data:**

| student_id | course_id | enrollment_date |
|------------|-----------|-----------------|
| 101 | 201 | 2024-01-15 |
| 101 | 202 | 2024-01-16 |
| 102 | 201 | 2024-01-17 |

**Key Points:**
- Combination of `student_id` AND `course_id` must be unique
- Same student can enroll in multiple courses
- Same course can have multiple students
- But same student cannot enroll in same course twice

**Valid vs Invalid:**

```sql
-- VALID: Different course for same student
INSERT INTO enrollments VALUES (101, 203, '2024-01-18');

-- INVALID: Duplicate combination
INSERT INTO enrollments VALUES (101, 201, '2024-01-20');
```

---

### Natural vs Surrogate Keys

**Natural Key**: Real-world identifier (email, SSN, ISBN)
**Surrogate Key**: Artificial identifier (auto-increment ID)

| Aspect | Natural Key | Surrogate Key |
|--------|-------------|---------------|
| **Meaning** | Has business meaning | No business meaning |
| **Example** | email, SSN | id (1, 2, 3...) |
| **Stability** | May change | Never changes |
| **Length** | Often longer | Usually short |
| **Recommended** | Rare | Most common |

**Example Comparison:**

```sql
-- Natural Key (email as primary key)
CREATE TABLE users_natural (
    email VARCHAR(100) PRIMARY KEY,  -- Natural key
    name VARCHAR(100)
);

-- Surrogate Key (auto-increment ID)
CREATE TABLE users_surrogate (
    user_id INT AUTO_INCREMENT PRIMARY KEY,  -- Surrogate key
    email VARCHAR(100) UNIQUE,
    name VARCHAR(100)
);
```

**Recommendation**: Use surrogate keys for flexibility and performance.

---

## Foreign Key

### What is a Foreign Key

A **Foreign Key** is a column (or set of columns) that creates a link between two tables by referencing the primary key of another table.

**Purpose:**
- Enforce referential integrity
- Maintain relationships between tables
- Prevent orphaned records

**Characteristics:**
- References primary key in another table (or same table)
- Can contain NULL values (optional relationship)
- Can have duplicate values (many-to-one relationship)

---

### Creating Foreign Keys

**Example:**

```sql
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(100)
);

CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    name VARCHAR(100),
    dept_id INT,
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
);
```

**departments table:**

| dept_id | dept_name |
|---------|-----------|
| 10 | Sales |
| 20 | IT |

**employees table:**

| emp_id | name | dept_id |
|--------|------|---------|
| 1 | Alice | 10 |
| 2 | Bob | 20 |
| 3 | Carol | 10 |

**Key Points:**
- `dept_id` in employees references `dept_id` in departments
- Cannot insert employee with non-existent `dept_id`

---

### Referential Integrity

**Referential Integrity** ensures that foreign key values always reference valid primary key values.

**Rules Enforced:**
1. Cannot insert invalid foreign key value
2. Cannot delete referenced primary key (by default)
3. Cannot update referenced primary key (by default)

**Example - Violations:**

```sql
-- ERROR: dept_id 99 doesn't exist in departments
INSERT INTO employees VALUES (4, 'David', 99);

-- ERROR: dept_id 10 is referenced by employees
DELETE FROM departments WHERE dept_id = 10;
```

---

### Cascade Operations

Define behavior when referenced row is updated or deleted.

**Options:**
- **CASCADE**: Automatically update/delete child rows
- **SET NULL**: Set foreign key to NULL
- **SET DEFAULT**: Set foreign key to default value
- **RESTRICT**: Prevent operation (default)
- **NO ACTION**: Same as RESTRICT

**Example:**

```sql
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    name VARCHAR(100),
    dept_id INT,
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);
```

**Before:**

**departments:**
| dept_id | dept_name |
|---------|-----------|
| 10 | Sales |

**employees:**
| emp_id | name | dept_id |
|--------|------|---------|
| 1 | Alice | 10 |
| 2 | Bob | 10 |

**Operation:**
```sql
DELETE FROM departments WHERE dept_id = 10;
```

**After (with CASCADE):**

**departments:**
| dept_id | dept_name |
|---------|-----------|
| *(empty)* |

**employees:**
| emp_id | name | dept_id |
|--------|------|---------|
| *(empty)* |

*Explanation:* Deleting department 10 automatically deletes all employees in that department.

---

## Constraints

Constraints are rules enforced on table columns to ensure data quality and integrity.

### UNIQUE Constraint

Ensures all values in a column (or set of columns) are unique.

**Difference from Primary Key:**
- Can have multiple UNIQUE constraints per table
- Can contain NULL values (usually one NULL allowed)

**Example:**

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE,
    username VARCHAR(50) UNIQUE
);
```

**Sample Data:**

| user_id | email | username |
|---------|-------|----------|
| 1 | alice@example.com | alice123 |
| 2 | bob@example.com | bob456 |

**Valid Operations:**

```sql
-- VALID: All unique
INSERT INTO users VALUES (3, 'carol@example.com', 'carol789');

-- VALID: NULL is allowed
INSERT INTO users VALUES (4, NULL, 'dave101');
```

**Invalid Operations:**

```sql
-- ERROR: Duplicate email
INSERT INTO users VALUES (5, 'alice@example.com', 'alice999');

-- ERROR: Duplicate username
INSERT INTO users VALUES (6, 'new@example.com', 'alice123');
```

---

### CHECK Constraint

Enforces a condition on column values.

**Example:**

```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10, 2) CHECK (price > 0),
    stock INT CHECK (stock >= 0),
    category VARCHAR(50) CHECK (category IN ('Electronics', 'Clothing', 'Food'))
);
```

**Valid Data:**

| product_id | name | price | stock | category |
|------------|------|-------|-------|----------|
| 1 | Laptop | 999.99 | 10 | Electronics |
| 2 | Shirt | 29.99 | 0 | Clothing |

**Invalid Operations:**

```sql
-- ERROR: price must be > 0
INSERT INTO products VALUES (3, 'Free Item', 0, 5, 'Food');

-- ERROR: stock cannot be negative
INSERT INTO products VALUES (4, 'Widget', 19.99, -5, 'Electronics');

-- ERROR: invalid category
INSERT INTO products VALUES (5, 'Book', 15.99, 10, 'Books');
```

---

### NOT NULL Constraint

Ensures a column cannot contain NULL values.

**Example:**

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    shipping_address VARCHAR(200)  -- NULL allowed
);
```

**Valid Data:**

| order_id | customer_id | order_date | shipping_address |
|----------|-------------|------------|------------------|
| 1 | 101 | 2024-01-15 | 123 Main St |
| 2 | 102 | 2024-01-16 | NULL |

**Invalid Operations:**

```sql
-- ERROR: customer_id cannot be NULL
INSERT INTO orders VALUES (3, NULL, '2024-01-17', '456 Oak Ave');

-- ERROR: order_date cannot be NULL
INSERT INTO orders VALUES (4, 103, NULL, '789 Elm St');
```

---

### DEFAULT Constraint

Provides a default value when no value is specified.

**Example:**

```sql
CREATE TABLE tasks (
    task_id INT PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    status VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    priority INT DEFAULT 3
);
```

**Insert without specifying defaults:**

```sql
INSERT INTO tasks (task_id, title) VALUES (1, 'Complete report');
```

**Result:**

| task_id | title | status | created_at | priority |
|---------|-------|--------|------------|----------|
| 1 | Complete report | pending | 2024-02-05 10:30:00 | 3 |

*Explanation:* Default values automatically filled for status, created_at, and priority.

---

## Normalization

### What is Normalization

**Normalization** is the process of organizing data to reduce redundancy and improve data integrity.

**Goals:**
- Eliminate duplicate data
- Ensure data dependencies make sense
- Reduce data anomalies (insertion, update, deletion)

**Trade-offs:**
- **Pros**: Data integrity, storage efficiency, easier updates
- **Cons**: More complex queries, more joins, potential performance impact

---

### First Normal Form (1NF)

**Rules:**
1. Each column contains atomic (indivisible) values
2. Each column contains values of the same type
3. Each row is unique
4. No repeating groups

**Violation Example (NOT in 1NF):**

| student_id | name | courses |
|------------|------|---------|
| 1 | Alice | Math, Physics, Chemistry |
| 2 | Bob | Math, English |

**Problems:**
- `courses` column contains multiple values (not atomic)
- Cannot easily query students taking specific course
- Difficult to add/remove courses

**Converted to 1NF:**

| student_id | name | course |
|------------|------|--------|
| 1 | Alice | Math |
| 1 | Alice | Physics |
| 1 | Alice | Chemistry |
| 2 | Bob | Math |
| 2 | Bob | English |

**Result:**
- Each cell contains single value
- Easy to query and modify
- Still has redundancy (name repeated)

---

### Second Normal Form (2NF)

**Rules:**
1. Must be in 1NF
2. No partial dependencies (all non-key columns depend on entire primary key)

**Applies to tables with composite primary keys.**

**Violation Example (NOT in 2NF):**

| student_id | course_id | student_name | instructor |
|------------|-----------|--------------|------------|
| 1 | 101 | Alice | Dr. Smith |
| 1 | 102 | Alice | Dr. Jones |
| 2 | 101 | Bob | Dr. Smith |

**Problems:**
- `student_name` depends only on `student_id` (partial dependency)
- `instructor` depends only on `course_id` (partial dependency)
- Redundant data

**Converted to 2NF:**

**students table:**

| student_id | student_name |
|------------|--------------|
| 1 | Alice |
| 2 | Bob |

**courses table:**

| course_id | instructor |
|-----------|------------|
| 101 | Dr. Smith |
| 102 | Dr. Jones |

**enrollments table:**

| student_id | course_id |
|------------|-----------|
| 1 | 101 |
| 1 | 102 |
| 2 | 101 |

**Result:**
- No partial dependencies
- Each table has data dependent on entire primary key
- Less redundancy

---

### Third Normal Form (3NF)

**Rules:**
1. Must be in 2NF
2. No transitive dependencies (non-key columns depend only on primary key, not on other non-key columns)

**Violation Example (NOT in 3NF):**

| employee_id | name | dept_id | dept_name | dept_location |
|-------------|------|---------|-----------|---------------|
| 1 | Alice | 10 | Sales | New York |
| 2 | Bob | 20 | IT | Boston |
| 3 | Carol | 10 | Sales | New York |

**Problems:**
- `dept_name` depends on `dept_id`, not directly on `employee_id` (transitive dependency)
- `dept_location` depends on `dept_id` (transitive dependency)
- Department info repeated for each employee

**Converted to 3NF:**

**employees table:**

| employee_id | name | dept_id |
|-------------|------|---------|
| 1 | Alice | 10 |
| 2 | Bob | 20 |
| 3 | Carol | 10 |

**departments table:**

| dept_id | dept_name | dept_location |
|---------|-----------|---------------|
| 10 | Sales | New York |
| 20 | IT | Boston |

**Result:**
- No transitive dependencies
- Each non-key column depends only on primary key
- Department info stored once

---

### Denormalization

**Denormalization** is intentionally adding redundancy to improve read performance.

**When to Denormalize:**
- Read-heavy applications
- Performance is critical
- Complex joins are expensive

**Example:**

**Normalized (3NF):**

**orders table:**
| order_id | customer_id | order_date |
|----------|-------------|------------|
| 1 | 101 | 2024-01-15 |

**customers table:**
| customer_id | name | city |
|-------------|------|------|
| 101 | Alice | NYC |

**Denormalized (for performance):**

| order_id | customer_id | customer_name | customer_city | order_date |
|----------|-------------|---------------|---------------|------------|
| 1 | 101 | Alice | NYC | 2024-01-15 |

**Trade-off:**
- ✅ Faster queries (no joins needed)
- ❌ Redundant data
- ❌ Update anomalies (must update customer info in multiple places)

---

## Relationships

Relationships define how tables are connected.

### One-to-One (1:1)

**Definition:** Each row in Table A relates to exactly one row in Table B, and vice versa.

**Use Cases:**
- Split large table for organization
- Separate sensitive data
- Optional attributes

**Example:**

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(50),
    email VARCHAR(100)
);

CREATE TABLE user_profiles (
    user_id INT PRIMARY KEY,
    bio TEXT,
    avatar_url VARCHAR(200),
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```

**users table:**

| user_id | username | email |
|---------|----------|-------|
| 1 | alice123 | alice@example.com |
| 2 | bob456 | bob@example.com |

**user_profiles table:**

| user_id | bio | avatar_url |
|---------|-----|------------|
| 1 | Software developer | /img/alice.jpg |
| 2 | Designer | /img/bob.jpg |

**Diagram:**
```
users (1) ←→ (1) user_profiles
```

*Explanation:* Each user has exactly one profile. Each profile belongs to exactly one user.

---

### One-to-Many (1:N)

**Definition:** Each row in Table A can relate to multiple rows in Table B, but each row in Table B relates to only one row in Table A.

**Use Cases:**
- Most common relationship type
- Parent-child relationships
- Hierarchical data

**Example:**

```sql
CREATE TABLE authors (
    author_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE books (
    book_id INT PRIMARY KEY,
    title VARCHAR(200),
    author_id INT,
    FOREIGN KEY (author_id) REFERENCES authors(author_id)
);
```

**authors table:**

| author_id | name |
|-----------|------|
| 1 | J.K. Rowling |
| 2 | Stephen King |

**books table:**

| book_id | title | author_id |
|---------|-------|-----------|
| 101 | Harry Potter 1 | 1 |
| 102 | Harry Potter 2 | 1 |
| 103 | The Shining | 2 |
| 104 | It | 2 |

**Diagram:**
```
authors (1) ←→ (many) books
```

*Explanation:* One author can write many books. Each book has one author.

---

### Many-to-Many (M:N)

**Definition:** Each row in Table A can relate to multiple rows in Table B, and vice versa.

**Implementation:** Requires a **junction table** (also called bridge table or associative table).

**Use Cases:**
- Students and courses
- Products and orders
- Tags and posts

**Example:**

```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE courses (
    course_id INT PRIMARY KEY,
    course_name VARCHAR(100)
);

-- Junction table
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    enrollment_date DATE,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

**students table:**

| student_id | name |
|------------|------|
| 1 | Alice |
| 2 | Bob |

**courses table:**

| course_id | course_name |
|-----------|-------------|
| 101 | Math |
| 102 | Physics |

**enrollments table (junction):**

| student_id | course_id | enrollment_date |
|------------|-----------|-----------------|
| 1 | 101 | 2024-01-15 |
| 1 | 102 | 2024-01-16 |
| 2 | 101 | 2024-01-17 |

**Diagram:**
```
students (many) ←→ enrollments ←→ (many) courses
```

*Explanation:* 
- Alice enrolled in Math and Physics
- Bob enrolled in Math
- Math has Alice and Bob
- Physics has Alice only

**Query Example:**

```sql
-- Find all courses for a student
SELECT c.course_name
FROM courses c
JOIN enrollments e ON c.course_id = e.course_id
WHERE e.student_id = 1;
```

**Result:**

| course_name |
|-------------|
| Math |
| Physics |

---

## Best Practices

### Keys
1. **Always define primary keys** - Essential for data integrity
2. **Use surrogate keys** (auto-increment IDs) for flexibility
3. **Index foreign keys** - Improves join performance
4. **Name foreign keys descriptively** - Include both table names

### Constraints
5. **Use NOT NULL** for required fields - Prevents data quality issues
6. **Add CHECK constraints** for business rules - Validates data at database level
7. **Use UNIQUE** for natural identifiers - Email, username, etc.
8. **Consider DEFAULT values** - Simplifies inserts

### Normalization
9. **Normalize to 3NF** as a starting point - Reduces redundancy
10. **Denormalize strategically** - Only when performance requires it
11. **Document denormalization decisions** - Explain why and where
12. **Keep audit trail** of changes - Track data modifications

### Relationships
13. **Use junction tables** for many-to-many - Proper design pattern
14. **Consider cascade carefully** - Understand impact of deletes
15. **Name junction tables clearly** - Combine both entity names

### General
16. **Use consistent naming conventions** - Improves maintainability
17. **Document your schema** - Include ERD diagrams
18. **Plan for scalability** - Consider future growth
19. **Balance normalization and performance** - Find the right trade-off
20. **Test constraint behavior** - Verify integrity rules work correctly

---

## Common Interview Questions

### Q1: What's the difference between Primary Key and Unique constraint?

**Answer:**

| Feature | Primary Key | Unique Constraint |
|---------|-------------|-------------------|
| **NULL values** | Not allowed | Allowed (usually one) |
| **Per table** | Only one | Multiple allowed |
| **Purpose** | Uniquely identify rows | Enforce uniqueness |
| **Index** | Automatically creates clustered index | Creates non-clustered index |
| **Foreign Key reference** | Yes | Yes (can be referenced) |

**Example:**

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,          -- One primary key
    email VARCHAR(100) UNIQUE,         -- Multiple unique constraints
    username VARCHAR(50) UNIQUE,
    phone VARCHAR(20) UNIQUE
);
```

---

### Q2: Explain normalization with an example

**Answer:**

**Unnormalized Table:**

| order_id | customer | customer_email | products |
|----------|----------|----------------|----------|
| 1 | Alice | alice@example.com | Laptop, Mouse |
| 2 | Bob | bob@example.com | Keyboard |

**Problems:**
- Products not atomic (1NF violation)
- Customer info repeated (redundancy)

**After Normalization (3NF):**

**customers:**
| customer_id | name | email |
|-------------|------|-------|
| 1 | Alice | alice@example.com |
| 2 | Bob | bob@example.com |

**orders:**
| order_id | customer_id |
|----------|-------------|
| 1 | 1 |
| 2 | 2 |

**products:**
| product_id | name |
|------------|------|
| 1 | Laptop |
| 2 | Mouse |
| 3 | Keyboard |

**order_items:**
| order_id | product_id |
|----------|------------|
| 1 | 1 |
| 1 | 2 |
| 2 | 3 |

---

### Q3: How do you implement a many-to-many relationship?

**Answer:**

Use a **junction table** with foreign keys to both tables.

**Example - Students and Courses:**

```sql
-- Entity tables
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE courses (
    course_id INT PRIMARY KEY,
    course_name VARCHAR(100)
);

-- Junction table
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    grade CHAR(1),
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

**Why junction table?**
- Cannot store multiple course IDs in student table
- Cannot store multiple student IDs in course table
- Junction table stores the relationship with optional attributes (grade)

---

### Q4: When would you denormalize a database?

**Answer:**

**Denormalize when:**
1. **Read performance is critical** - Heavy SELECT operations
2. **Complex joins are expensive** - Many tables involved
3. **Data rarely changes** - Update anomalies less concerning
4. **Reporting/analytics** - Data warehouse scenarios

**Example:**

Instead of joining every time:
```sql
-- Normalized (requires join)
SELECT o.order_id, c.customer_name, c.city
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;
```

Denormalize for speed:
```sql
-- Denormalized (no join needed)
CREATE TABLE orders_denorm (
    order_id INT,
    customer_id INT,
    customer_name VARCHAR(100),  -- Redundant
    customer_city VARCHAR(50)    -- Redundant
);
```

---

### Q5: What are the CASCADE options in foreign keys?

**Answer:**

| Option | Behavior on DELETE/UPDATE |
|--------|---------------------------|
| **CASCADE** | Automatically delete/update child rows |
| **SET NULL** | Set foreign key to NULL |
| **SET DEFAULT** | Set foreign key to default value |
| **RESTRICT** | Prevent operation (raise error) |
| **NO ACTION** | Same as RESTRICT |

**Example:**

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        ON DELETE CASCADE      -- Delete orders when customer deleted
        ON UPDATE CASCADE      -- Update order's customer_id if customer_id changes
);
```

**Use cases:**
- **CASCADE**: Parent-child dependency (orders belong to customer)
- **SET NULL**: Optional relationship (employee's manager)
- **RESTRICT**: Prevent accidental deletes (prevent deleting customer with orders)

---

## Summary

### Keys Summary

| Key Type | Purpose | Uniqueness | NULL Values |
|----------|---------|------------|-------------|
| **Primary Key** | Uniquely identify rows | Always unique | Not allowed |
| **Foreign Key** | Link tables | Can duplicate | Allowed |
| **Unique Key** | Enforce uniqueness | Unique | Allowed (one) |

### Normalization Summary

| Normal Form | Key Rule | Eliminates |
|-------------|----------|------------|
| **1NF** | Atomic values, unique rows | Repeating groups |
| **2NF** | 1NF + no partial dependencies | Partial dependencies |
| **3NF** | 2NF + no transitive dependencies | Transitive dependencies |

### Relationship Summary

| Type | Description | Implementation |
|------|-------------|----------------|
| **1:1** | One-to-one | Foreign key with UNIQUE constraint |
| **1:N** | One-to-many | Foreign key in "many" table |
| **M:N** | Many-to-many | Junction table with two foreign keys |

### Key Takeaways

1. **Primary keys** uniquely identify records - always define them
2. **Foreign keys** maintain referential integrity - link tables properly
3. **Constraints** enforce data quality - use NOT NULL, CHECK, UNIQUE
4. **Normalize to 3NF** - reduce redundancy and anomalies
5. **Denormalize carefully** - only for performance when needed
6. **Junction tables** implement many-to-many relationships
7. **CASCADE options** control referential actions - use appropriately

**Master these fundamentals for solid database design!** 🎯
