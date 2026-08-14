# SQL Interview Preparation — 6+ Years Java Backend

Focused SQL roadmap for a **6+ years experienced Java / Spring Boot backend developer**.

### Priority

- 🔴 **P0** — Must Know
- 🟠 **P1** — Should Know
- 🔵 **Java/Spring Specific** — Must Know for Backend Interviews

---

# 🔴 P0 — Must Know

## 01. SQL Basics

- [ ] SELECT
- [ ] WHERE
- [ ] ORDER BY
- [ ] DISTINCT
- [ ] LIMIT / OFFSET
- [ ] NULL
- [ ] IN
- [ ] BETWEEN
- [ ] LIKE
- [ ] CASE
- [ ] COALESCE

---

## 02. Aggregation

- [ ] COUNT
- [ ] SUM
- [ ] AVG
- [ ] MIN
- [ ] MAX
- [ ] GROUP BY
- [ ] HAVING
- [ ] WHERE vs HAVING
- [ ] NULL with aggregate functions

---

## 03. Joins ⭐

- [ ] INNER JOIN
- [ ] LEFT JOIN
- [ ] RIGHT JOIN
- [ ] FULL OUTER JOIN
- [ ] SELF JOIN
- [ ] CROSS JOIN
- [ ] Multiple JOINs
- [ ] JOIN conditions
- [ ] JOIN vs EXISTS
- [ ] JOIN vs Subquery

### Practice

- [ ] Employees without departments
- [ ] Departments without employees
- [ ] Customers without orders
- [ ] Employees earning more than managers
- [ ] Joining 3+ tables

---

## 04. Subqueries

- [ ] Scalar Subquery
- [ ] Correlated Subquery
- [ ] Non-correlated Subquery
- [ ] Subquery in WHERE
- [ ] Subquery in SELECT
- [ ] Subquery in FROM
- [ ] IN vs EXISTS
- [ ] NOT IN vs NOT EXISTS

---

## 05. Window Functions ⭐

- [ ] OVER()
- [ ] PARTITION BY
- [ ] ROW_NUMBER()
- [ ] RANK()
- [ ] DENSE_RANK()
- [ ] LEAD()
- [ ] LAG()
- [ ] Running Total
- [ ] Top N per Group
- [ ] RANK vs DENSE_RANK vs ROW_NUMBER

---

## 06. CTE

- [ ] Basic CTE
- [ ] Multiple CTEs
- [ ] CTE with JOIN
- [ ] CTE with Aggregation
- [ ] Recursive CTE — Basic Understanding
- [ ] CTE vs Subquery

---

## 07. Indexes ⭐⭐⭐

- [ ] What is an Index?
- [ ] Why Indexes Improve Performance
- [ ] B-Tree Index — Basic Understanding
- [ ] Clustered Index
- [ ] Non-Clustered Index
- [ ] Composite Index
- [ ] Composite Index Column Order
- [ ] Index Selectivity
- [ ] Cardinality
- [ ] Covering Index
- [ ] Index Scan
- [ ] Index Seek
- [ ] Full Table Scan
- [ ] Why an Index May Not Be Used
- [ ] When NOT to Create an Index
- [ ] Too Many Indexes

---

## 08. Transactions & ACID ⭐⭐⭐

### Transactions

- [ ] Transaction
- [ ] COMMIT
- [ ] ROLLBACK
- [ ] SAVEPOINT
- [ ] Transaction Boundaries
- [ ] Auto Commit

### ACID

- [ ] Atomicity
- [ ] Consistency
- [ ] Isolation
- [ ] Durability

---

## 09. Isolation & Concurrency ⭐⭐⭐

### Isolation Levels

- [ ] Read Uncommitted
- [ ] Read Committed
- [ ] Repeatable Read
- [ ] Serializable

### Concurrency Problems

- [ ] Dirty Read
- [ ] Non-Repeatable Read
- [ ] Phantom Read
- [ ] Lost Update

### Locking

- [ ] Shared Lock
- [ ] Exclusive Lock
- [ ] Row-Level Lock
- [ ] Pessimistic Locking
- [ ] Optimistic Locking
- [ ] Deadlock
- [ ] Blocking

---

## 10. Query Execution & Optimization ⭐⭐⭐

### Query Processing

- [ ] Logical Query Processing Order
- [ ] FROM
- [ ] JOIN
- [ ] WHERE
- [ ] GROUP BY
- [ ] HAVING
- [ ] SELECT
- [ ] DISTINCT
- [ ] ORDER BY
- [ ] LIMIT / OFFSET

### Execution Plan

- [ ] Execution Plan
- [ ] EXPLAIN
- [ ] EXPLAIN ANALYZE
- [ ] Table Scan
- [ ] Index Scan
- [ ] Index Seek
- [ ] Join Execution
- [ ] Sort
- [ ] Query Cost
- [ ] Cardinality Estimation

### Optimization

- [ ] Identify Slow Queries
- [ ] Optimize JOINs
- [ ] Optimize Indexes
- [ ] Avoid SELECT *
- [ ] Avoid Unnecessary DISTINCT
- [ ] Avoid Unnecessary JOINs
- [ ] Avoid Functions on Indexed Columns
- [ ] Query Rewriting

---

## 11. Pagination

- [ ] LIMIT / OFFSET Pagination
- [ ] Problems with Large OFFSET
- [ ] Keyset Pagination
- [ ] Cursor Pagination
- [ ] Pagination with Indexes

---

## 12. Database Design

### Keys

- [ ] Primary Key
- [ ] Foreign Key
- [ ] Composite Key
- [ ] Natural Key — Basic Understanding
- [ ] Surrogate Key — Basic Understanding

### Relationships

- [ ] One-to-One
- [ ] One-to-Many
- [ ] Many-to-One
- [ ] Many-to-Many
- [ ] Junction Table

### Integrity

- [ ] Referential Integrity
- [ ] Cascade
- [ ] ON DELETE
- [ ] ON UPDATE

---

## 13. Normalization

- [ ] 1NF
- [ ] 2NF
- [ ] 3NF
- [ ] BCNF — Basic Understanding
- [ ] Normalization vs Denormalization
- [ ] When to Denormalize

---

## 14. SQL Interview Problems ⭐⭐⭐

### Basic / Intermediate

- [ ] Second Highest Salary
- [ ] Nth Highest Salary
- [ ] Highest Salary Per Department
- [ ] Top N Salaries Per Department
- [ ] Employees Earning More Than Manager
- [ ] Find Duplicate Records
- [ ] Delete Duplicate Records
- [ ] Customers Without Orders
- [ ] Departments Without Employees
- [ ] Find Missing IDs

### Advanced

- [ ] Latest Record Per Customer
- [ ] First Record Per Customer
- [ ] Running Total
- [ ] Ranking Problems
- [ ] Consecutive Records
- [ ] Gaps and Islands
- [ ] Month-over-Month Comparison
- [ ] Year-over-Year Comparison
- [ ] Self-Join Problems
- [ ] Hierarchical Queries

---

# 🟠 P1 — Should Know

## 15. DDL / DML

- [ ] CREATE
- [ ] ALTER
- [ ] DROP
- [ ] TRUNCATE
- [ ] INSERT
- [ ] UPDATE
- [ ] DELETE
- [ ] DELETE vs TRUNCATE vs DROP
- [ ] INSERT INTO SELECT
- [ ] UPSERT — Basic Understanding

---

## 16. Views

- [ ] What is a View?
- [ ] Creating Views
- [ ] View vs Table
- [ ] Materialized View
- [ ] View vs Materialized View
- [ ] Basic Use Cases

---

## 17. Partitioning

- [ ] What is Partitioning?
- [ ] Range Partitioning
- [ ] Hash Partitioning
- [ ] List Partitioning
- [ ] Partition Pruning
- [ ] Partitioning vs Sharding
- [ ] When Partitioning is Useful

---

## 18. Replication

- [ ] Primary / Replica
- [ ] Read Replica
- [ ] Synchronous Replication
- [ ] Asynchronous Replication
- [ ] Replication Lag
- [ ] Failover
- [ ] High Availability — Basic Understanding

---

## 19. Sharding

- [ ] What is Sharding?
- [ ] Shard Key
- [ ] Horizontal Scaling
- [ ] Cross-Shard Query Challenges
- [ ] Partitioning vs Sharding

---

## 20. SQL Security

- [ ] SQL Injection
- [ ] Prepared Statements
- [ ] Parameterized Queries
- [ ] Database Users
- [ ] Database Roles
- [ ] GRANT
- [ ] REVOKE
- [ ] Least Privilege

---

# 🔵 Java / Spring Specific — P0

## 21. JDBC

- [ ] JDBC Basics
- [ ] Connection
- [ ] Statement
- [ ] PreparedStatement
- [ ] CallableStatement — Basic Understanding
- [ ] ResultSet
- [ ] Connection Pooling
- [ ] HikariCP
- [ ] JDBC Transactions
- [ ] Batch Operations
- [ ] Connection Lifecycle

---

## 22. JPA / Hibernate + SQL ⭐⭐⭐

- [ ] JPQL vs SQL
- [ ] Native Queries
- [ ] Entity Mapping
- [ ] Lazy Loading
- [ ] Eager Loading
- [ ] N+1 Query Problem
- [ ] Fetch Join
- [ ] Persistence Context
- [ ] First-Level Cache
- [ ] Dirty Checking
- [ ] @Transactional
- [ ] Optimistic Locking
- [ ] Pessimistic Locking
- [ ] Batch Fetching
- [ ] JDBC Batching
- [ ] Query Performance
- [ ] SQL Generated by Hibernate
- [ ] Hibernate Query Optimization

---

# ❌ Low Priority — Awareness Only

These don't need dedicated preparation for a typical 6–7 year Java backend interview unless your project specifically uses them.

- [ ] 4NF
- [ ] 5NF
- [ ] Advanced Stored Procedure Programming
- [ ] Advanced Trigger Programming
- [ ] Advanced Database Functions
- [ ] Sequence Internals
- [ ] Advanced Replication Internals
- [ ] Advanced Sharding Algorithms
- [ ] Two-Phase Commit Internals
- [ ] CDC Internals
- [ ] Deep Database Engine Internals
- [ ] Vendor-Specific Obscure Features
- [ ] Advanced Procedural SQL

---

# 🎯 Final Interview Readiness

I should be able to confidently answer:

- [ ] How does a JOIN work?
- [ ] INNER JOIN vs LEFT JOIN?
- [ ] JOIN vs EXISTS?
- [ ] WHERE vs HAVING?
- [ ] RANK vs DENSE_RANK vs ROW_NUMBER?
- [ ] How do window functions work?
- [ ] What is a correlated subquery?
- [ ] What is a CTE?
- [ ] How does an index work?
- [ ] Why is an index not being used?
- [ ] How do you design a composite index?
- [ ] What is a covering index?
- [ ] What is ACID?
- [ ] Explain all transaction isolation levels.
- [ ] What is a dirty read?
- [ ] What is a phantom read?
- [ ] Optimistic vs pessimistic locking?
- [ ] How does a deadlock happen?
- [ ] How do you troubleshoot a slow query?
- [ ] How do you read an execution plan?
- [ ] OFFSET vs keyset pagination?
- [ ] How do you solve N+1 in Hibernate?
- [ ] How does @Transactional work with database transactions?
- [ ] How does Hibernate generate SQL?
- [ ] How would you optimize a database-heavy API?
- [ ] How would you scale a database?

---

# 📊 Recommended Priority

| Area | Priority | Depth |
|---|---|---|
| SQL Basics | P0 | Strong |
| Aggregation | P0 | Strong |
| Joins | P0 | Very Strong |
| Subqueries | P0 | Strong |
| Window Functions | P0 | Strong |
| CTE | P0 | Strong |
| Indexes | P0 | Very Strong |
| Transactions / ACID | P0 | Very Strong |
| Isolation / Locks | P0 | Very Strong |
| Query Optimization | P0 | Very Strong |
| Pagination | P0 | Strong |
| Database Design | P0 | Strong |
| Normalization | P0 | Medium |
| SQL Problems | P0 | Strong |
| DDL / DML | P1 | Medium |
| Views | P1 | Basic |
| Partitioning | P1 | Concept |
| Replication | P1 | Concept |
| Sharding | P1 | Concept |
| SQL Security | P1 | Medium |
| JDBC | P0 | Strong |
| JPA / Hibernate SQL | P0 | Very Strong |

---

# 🚀 Recommended Study Order

```text
SQL Basics
    ↓
Aggregation
    ↓
JOINs
    ↓
Subqueries
    ↓
Window Functions
    ↓
CTEs
    ↓
Indexes
    ↓
Transactions & ACID
    ↓
Isolation & Locking
    ↓
Query Execution
    ↓
Query Optimization
    ↓
Pagination
    ↓
Database Design
    ↓
Normalization
    ↓
SQL Problems
    ↓
JDBC
    ↓
JPA / Hibernate + SQL
    ↓
Partitioning / Replication / Sharding