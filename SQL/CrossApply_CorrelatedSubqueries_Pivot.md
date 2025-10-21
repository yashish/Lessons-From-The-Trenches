`CROSS APPLY` is a powerful SQL operator used in SQL Server and Azure SQL Database to **join a table with a table-valued function (TVF)** or **a subquery that depends on columns of the outer table**. It acts like a **correlated join**, evaluating the right-side expression **for each row** of the left-side table.

---

### 🔧 Syntax:

```sql
SELECT ...
FROM OuterTable o
CROSS APPLY (
    SELECT ...
    FROM SomeTableOrFunction f
    WHERE f.SomeColumn = o.SomeColumn
) AS derived
```

---

### ✅ When to Use `CROSS APPLY`

* When you want to **invoke a table-valued function** for each row of a table.
* When the **right-side query depends on columns from the left-side table**.
* When you want to **filter or transform** row-by-row with logic that varies by input row.
* Useful as an alternative to **correlated subqueries** or **cursor-based logic**.

---

### 📌 Example Scenario

Suppose you have:

* A `Customers` table.
* An `Orders` table.
  You want to fetch the **latest order** for each customer.

---

### 🧩 Schema

```sql
CREATE TABLE Customers (
    CustomerID INT,
    CustomerName VARCHAR(100)
);

CREATE TABLE Orders (
    OrderID INT,
    CustomerID INT,
    OrderDate DATE,
    Amount DECIMAL(10, 2)
);
```

---

### ✅ Query Using `CROSS APPLY` to Get Latest Order Per Customer

```sql
SELECT 
    c.CustomerID,
    c.CustomerName,
    o.OrderID,
    o.OrderDate,
    o.Amount
FROM 
    Customers c
CROSS APPLY (
    SELECT TOP 1 *
    FROM Orders o
    WHERE o.CustomerID = c.CustomerID
    ORDER BY o.OrderDate DESC
) o;
```

---

### 🔍 What’s Happening:

* For each row in `Customers`, the subquery on `Orders` is **re-evaluated**.
* It fetches only the **latest (TOP 1)** order by date.
* `CROSS APPLY` allows the subquery to **reference the outer table's column** (`c.CustomerID`).

---

### 🆚 `OUTER APPLY` vs `CROSS APPLY`

* `CROSS APPLY`: behaves like an **INNER JOIN** — excludes outer rows that return no results from the right side.
* `OUTER APPLY`: behaves like a **LEFT JOIN** — includes all outer rows, NULLs if the apply query returns nothing.

---

### 💡 Real-World Use Cases

* Call table-valued functions with arguments from each row (e.g., `CROSS APPLY dbo.fn_Split(c.CSV)`).
* Extract JSON/XML data from a column per row.
* Get top N related items (e.g., top 3 transactions per customer).
* Pagination logic where filters are row-specific.

Great! Here's a practical example where we use **`CROSS APPLY` or `OUTER APPLY`** with a **TVF that parses CSV strings**.

---

### 🎯 Scenario:

You have a table called `Users` with a column `FavoriteColors` that stores CSV strings like `'Red,Blue,Green'`. You want to **split those values into rows**, one per color, per user.

---

### ✅ Step 1: Sample Table

```sql
CREATE TABLE Users (
    UserID INT PRIMARY KEY,
    UserName VARCHAR(100),
    FavoriteColors VARCHAR(100)
);

INSERT INTO Users (UserID, UserName, FavoriteColors) VALUES
(1, 'Alice', 'Red,Blue,Green'),
(2, 'Bob', 'Yellow'),
(3, 'Charlie', NULL);  -- No favorite colors
```

---

### ✅ Step 2: Create a TVF to Split CSV

```sql
CREATE FUNCTION dbo.SplitCSV (@CSV NVARCHAR(MAX))
RETURNS @Result TABLE (Value NVARCHAR(100))
AS
BEGIN
    DECLARE @pos INT = 0, @nextPos INT, @val NVARCHAR(100)

    WHILE 1 = 1
    BEGIN
        SET @nextPos = CHARINDEX(',', @CSV, @pos + 1)
        IF @nextPos = 0 
            SET @nextPos = LEN(@CSV) + 1
        
        SET @val = LTRIM(RTRIM(SUBSTRING(@CSV, @pos + 1, @nextPos - @pos - 1)))
        IF @val <> '' INSERT INTO @Result (Value) VALUES (@val)
        
        IF @nextPos >= LEN(@CSV) BREAK
        SET @pos = @nextPos
    END

    RETURN
END
```

---

### ✅ Step 3: Use `OUTER APPLY` to Expand CSV Values

```sql
SELECT 
    u.UserID,
    u.UserName,
    Colors.Value AS FavoriteColor
FROM 
    Users u
OUTER APPLY dbo.SplitCSV(u.FavoriteColors) Colors;
```

---

### 🔍 What This Does:

* `dbo.SplitCSV` is invoked for each user.
* If `FavoriteColors` is `NULL`, `OUTER APPLY` ensures the user still appears in the result (with `NULL` for `FavoriteColor`).
* Each CSV item becomes a separate row.

---

### ✅ Output:

| UserID | UserName | FavoriteColor |
| ------ | -------- | ------------- |
| 1      | Alice    | Red           |
| 1      | Alice    | Blue          |
| 1      | Alice    | Green         |
| 2      | Bob      | Yellow        |
| 3      | Charlie  | NULL          |

---

### 🧠 Use Cases for This Pattern:

* Parsing CSV tags, emails, roles, product codes.
* Normalizing denormalized data for joins or filtering.
* Tokenizing log data, configurations, etc.

Would you like a SQL Server 2016+ version using `STRING_SPLIT()` instead?


Great! If you're using **SQL Server 2016 or later**, you can simplify CSV parsing using the built-in **`STRING_SPLIT()`** function.

---

### ⚠️ Important Notes about `STRING_SPLIT()`:

* It returns a **single-column** result named `value`.
* Prior to SQL Server 2022, **ordering is not guaranteed**, even if the original CSV was ordered.
* SQL Server 2022 introduced an optional `ordinal` column (if `STRING_SPLIT()` is called with `enable_ordinal = 1`).

---

### ✅ Step-by-Step Example Using `STRING_SPLIT()`

#### 🧩 Step 1: Sample Data

```sql
CREATE TABLE Users (
    UserID INT PRIMARY KEY,
    UserName VARCHAR(100),
    FavoriteColors VARCHAR(100)
);

INSERT INTO Users (UserID, UserName, FavoriteColors) VALUES
(1, 'Alice', 'Red,Blue,Green'),
(2, 'Bob', 'Yellow'),
(3, 'Charlie', NULL);  -- No favorite colors
```

---

### ✅ Step 2: Query Using `OUTER APPLY` + `STRING_SPLIT()`

```sql
SELECT 
    u.UserID,
    u.UserName,
    c.value AS FavoriteColor
FROM 
    Users u
OUTER APPLY STRING_SPLIT(u.FavoriteColors, ',') AS c;
```

---

### ✅ Output:

| UserID | UserName | FavoriteColor |
| ------ | -------- | ------------- |
| 1      | Alice    | Red           |
| 1      | Alice    | Blue          |
| 1      | Alice    | Green         |
| 2      | Bob      | Yellow        |
| 3      | Charlie  | NULL          |

---

### ✅ Optional: Preserve Order (SQL Server 2022+ Only)

```sql
-- Only works in SQL Server 2022+
SELECT 
    u.UserID,
    u.UserName,
    c.value AS FavoriteColor,
    c.ordinal
FROM 
    Users u
OUTER APPLY STRING_SPLIT(u.FavoriteColors, ',', 1) AS c
ORDER BY u.UserID, c.ordinal;
```

> The `ordinal` column preserves the original sequence of CSV items.

---

### 🚀 Summary

* ✅ Use `STRING_SPLIT()` for simple, fast parsing when order isn't important.
* 🧠 Still use `OUTER APPLY` to safely handle rows with `NULL` CSV values.
* 🔄 For pre-2016 versions or if ordering matters (before 2022), use a custom `SplitCSV` function.

Would you like a reusable `STRING_SPLIT` wrapper function that preserves order for older SQL Server versions?

A **correlated subquery** is a **subquery that depends on values from the outer query** to execute. Unlike a regular (uncorrelated) subquery, which can run independently, a **correlated subquery is evaluated once for each row** returned by the outer query.

---

### 🔍 Key Characteristics

* It **references a column from the outer query** inside the subquery.
* It is evaluated **row-by-row**, not just once.
* It is often used in the `WHERE`, `SELECT`, or `FROM` clause.

---

### ✅ Example: Find Customers with Their Latest Order Date

#### 🧩 Tables:

```sql
Customers (CustomerID, CustomerName)
Orders (OrderID, CustomerID, OrderDate)
```

---

#### 🧠 Correlated Subquery in `SELECT`

```sql
SELECT 
    c.CustomerID,
    c.CustomerName,
    (
        SELECT MAX(o.OrderDate)
        FROM Orders o
        WHERE o.CustomerID = c.CustomerID
    ) AS LatestOrderDate
FROM Customers c;
```

---

### 🔄 What's Happening Here:

* The subquery:

  ```sql
  SELECT MAX(o.OrderDate) FROM Orders o WHERE o.CustomerID = c.CustomerID
  ```

  cannot be run on its own meaningfully — it needs `c.CustomerID` from the **outer query**.
* SQL Server runs this subquery **once for each row** in `Customers`.

---

### ✅ Example: Correlated Subquery in `WHERE` Clause

> Find all customers who have placed at least one order.

```sql
SELECT CustomerID, CustomerName
FROM Customers c
WHERE EXISTS (
    SELECT 1
    FROM Orders o
    WHERE o.CustomerID = c.CustomerID
);
```

* The subquery **depends on `c.CustomerID`** and is evaluated per row.
* `EXISTS` is true if the subquery returns **any row** — more efficient than `IN` in many cases.

---

### ⚖️ Correlated vs. Non-Correlated Subquery

| Type               | Can run independently? | Evaluated per row? | Example Use Case                         |
| ------------------ | ---------------------- | ------------------ | ---------------------------------------- |
| **Correlated**     | ❌ No                   | ✅ Yes              | Latest order per customer                |
| **Non-correlated** | ✅ Yes                  | ❌ No               | Get customers from specific country code |

---

### 🧠 Performance Note:

Correlated subqueries can be **slow on large datasets** since they’re evaluated repeatedly. You can often **rewrite them using JOINs or APPLY** for better performance.

Would you like to see how this example can be rewritten using `OUTER APPLY` or a `JOIN` for optimization?

Perfect. Let’s take the same use case:
👉 **“Get each customer and their latest order date.”**

We’ll rewrite it **three ways**:

1. Using a **Correlated Subquery** (as baseline),
2. Rewriting it using **`OUTER APPLY`**,
3. Rewriting it using a **`LEFT JOIN` with a CTE or subquery**.

---

## ✅ 1. **Correlated Subquery (Baseline)**

```sql
SELECT 
    c.CustomerID,
    c.CustomerName,
    (
        SELECT MAX(o.OrderDate)
        FROM Orders o
        WHERE o.CustomerID = c.CustomerID
    ) AS LatestOrderDate
FROM Customers c;
```

* Subquery runs once per customer.
* OK for small datasets; performance degrades with size.

---

## ✅ 2. **Using `OUTER APPLY`**

```sql
SELECT 
    c.CustomerID,
    c.CustomerName,
    o.OrderDate AS LatestOrderDate
FROM 
    Customers c
OUTER APPLY (
    SELECT TOP 1 OrderDate
    FROM Orders o
    WHERE o.CustomerID = c.CustomerID
    ORDER BY OrderDate DESC
) o;
```

### 🔍 Why it's better:

* `OUTER APPLY` allows filtering and sorting **per row** like a correlated subquery.
* More **efficient** when you need more than one column from the latest order (e.g., `OrderID`, `Amount`).
* `TOP 1 + ORDER BY` fetches the latest row directly — no `MAX()` needed.

---

## ✅ 3. **Using `LEFT JOIN` + CTE**

This approach is good if you want **pure SQL compatibility** across DBMSs (e.g., PostgreSQL, MySQL).

```sql
WITH LatestOrders AS (
    SELECT 
        CustomerID,
        MAX(OrderDate) AS LatestOrderDate
    FROM Orders
    GROUP BY CustomerID
)
SELECT 
    c.CustomerID,
    c.CustomerName,
    lo.LatestOrderDate
FROM 
    Customers c
LEFT JOIN LatestOrders lo ON lo.CustomerID = c.CustomerID;
```

### 🔍 Why it’s good:

* Efficient for large datasets.
* Separation of aggregation (`MAX`) from join.
* Easier for optimization by SQL engine.

---

### 🧠 Summary Table

| Method              | SQL Feature Used            | Performance           | Best For                                    |
| ------------------- | --------------------------- | --------------------- | ------------------------------------------- |
| Correlated Subquery | `SELECT MAX(...) WHERE ...` | 👎 Slow on large data | Simple lookups with 1 column                |
| `OUTER APPLY`       | `TOP 1 ORDER BY` per row    | ✅ Efficient           | Getting multiple columns from latest row    |
| `LEFT JOIN + CTE`   | Aggregate + Join            | ✅ Efficient           | Aggregated value joins (e.g., `MAX`, `SUM`) |

---

Would you like to expand the `OUTER APPLY` to include full order details (e.g., `OrderID`, `Amount`, etc.) for the latest order as well?

Perfect 👍 — let’s extend the **`OUTER APPLY`** version so that we return **full details** of each customer’s *most recent order* (not just the date).

---

## 🧩 Example Tables

```sql
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    CustomerName VARCHAR(100)
);

CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT,
    OrderDate DATE,
    Amount DECIMAL(10, 2)
);
```

---

## ✅ Query Using `OUTER APPLY` to Get **Full Latest Order Details**

```sql
SELECT 
    c.CustomerID,
    c.CustomerName,
    o.OrderID,
    o.OrderDate,
    o.Amount
FROM 
    Customers c
OUTER APPLY (
    SELECT TOP 1 
        o2.OrderID,
        o2.OrderDate,
        o2.Amount
    FROM Orders o2
    WHERE o2.CustomerID = c.CustomerID
    ORDER BY o2.OrderDate DESC
) o
ORDER BY c.CustomerID;
```

---

### 🔍 How This Works

* For **each customer**, the subquery inside `OUTER APPLY` runs once.
* It fetches the **most recent order** by sorting in descending order by `OrderDate`.
* If a customer has **no orders**, `OUTER APPLY` ensures the customer **still appears** with `NULL` order details.

---

### ✅ Example Output

| CustomerID | CustomerName | OrderID | OrderDate  | Amount |
| ---------- | ------------ | ------- | ---------- | ------ |
| 1          | Alice        | 101     | 2024-09-10 | 250.00 |
| 2          | Bob          | 105     | 2024-07-15 | 99.99  |
| 3          | Charlie      | NULL    | NULL       | NULL   |

---

### ⚡ Why `OUTER APPLY` Is Ideal Here

* You can **return multiple columns** (not just the latest date).
* You can **sort or filter dynamically per row** (`TOP 1 ORDER BY`).
* Cleaner than correlated subqueries when you need *row-level logic*.

---

### 🧠 Bonus: `CROSS APPLY` version

If you only want customers **who actually have at least one order**, just switch to:

```sql
CROSS APPLY (
    SELECT TOP 1 
        o2.OrderID,
        o2.OrderDate,
        o2.Amount
    FROM Orders o2
    WHERE o2.CustomerID = c.CustomerID
    ORDER BY o2.OrderDate DESC
) o
```

That’s equivalent to an **INNER JOIN** — customers with no orders will be excluded.

---

Would you like me to show how this same logic would look using a **window function** (`ROW_NUMBER()`) instead of `APPLY`?
(It’s another very common and efficient modern SQL approach.)

Excellent — using a **window function** like `ROW_NUMBER()` is a modern, SQL-standard way to solve the *“latest record per group”* problem.
It’s often **more efficient** than `APPLY` when your dataset is large and indexes are properly tuned.

---

## 🧩 Problem

> Get each customer’s latest order **with full details** (OrderID, Date, Amount, etc.).

---

## ✅ Solution Using `ROW_NUMBER()`

```sql
WITH OrderedOrders AS (
    SELECT 
        o.OrderID,
        o.CustomerID,
        o.OrderDate,
        o.Amount,
        ROW_NUMBER() OVER (
            PARTITION BY o.CustomerID
            ORDER BY o.OrderDate DESC
        ) AS rn
    FROM Orders o
)
SELECT 
    c.CustomerID,
    c.CustomerName,
    o.OrderID,
    o.OrderDate,
    o.Amount
FROM 
    Customers c
LEFT JOIN OrderedOrders o
    ON o.CustomerID = c.CustomerID
    AND o.rn = 1  -- keep only the latest order per customer
ORDER BY c.CustomerID;
```

---

### 🔍 How It Works

1. **`ROW_NUMBER()`** assigns a sequence to each order per customer:

   * Starts at 1 for the most recent order (because we sort `DESC` by `OrderDate`).
2. The **CTE (`OrderedOrders`)** stores these ranked rows.
3. In the main query:

   * We **`LEFT JOIN`** customers to their “rank 1” (latest) order.
   * If a customer has no orders → still included (with `NULL` order columns).

---

### ✅ Example Output

| CustomerID | CustomerName | OrderID | OrderDate  | Amount |
| ---------- | ------------ | ------- | ---------- | ------ |
| 1          | Alice        | 101     | 2024-09-10 | 250.00 |
| 2          | Bob          | 105     | 2024-07-15 | 99.99  |
| 3          | Charlie      | NULL    | NULL       | NULL   |

---

### ⚖️ Comparison of Approaches

| Approach                           | SQL Feature                 | Performance            | Includes Customers with No Orders | Notes                    |
| ---------------------------------- | --------------------------- | ---------------------- | --------------------------------- | ------------------------ |
| **Correlated Subquery**            | `SELECT MAX(...) WHERE ...` | ❌ Slow on large data   | ✅ Yes                             | Simple but row-by-row    |
| **`OUTER APPLY`**                  | `TOP 1 ORDER BY`            | ✅ Good                 | ✅ Yes                             | Easy for per-row logic   |
| **`ROW_NUMBER()` Window Function** | `OVER (PARTITION BY …)`     | 🚀 Best for large sets | ✅ Yes                             | Most scalable + ANSI SQL |

---

### 🧠 Tip

If you only want customers **who have at least one order**, change the `LEFT JOIN` to an `INNER JOIN`.

---

Would you like me to show the **same pattern extended to get the top 3 latest orders per customer** using `ROW_NUMBER()` or `RANK()` next?

Perfect 👏 — let’s take this up a level and show how to get the **Top 3 latest orders per customer**, using **both `ROW_NUMBER()`** and **`RANK()`** window functions — and explain their differences.

---

## 🧩 Setup

We’ll assume the same tables:

```sql
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    CustomerName VARCHAR(100)
);

CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT,
    OrderDate DATE,
    Amount DECIMAL(10, 2)
);
```

---

## ✅ Example 1: Using `ROW_NUMBER()`

```sql
WITH RankedOrders AS (
    SELECT 
        o.OrderID,
        o.CustomerID,
        o.OrderDate,
        o.Amount,
        ROW_NUMBER() OVER (
            PARTITION BY o.CustomerID
            ORDER BY o.OrderDate DESC
        ) AS rn
    FROM Orders o
)
SELECT 
    c.CustomerID,
    c.CustomerName,
    r.OrderID,
    r.OrderDate,
    r.Amount
FROM 
    Customers c
LEFT JOIN RankedOrders r
    ON c.CustomerID = r.CustomerID
   AND r.rn <= 3   -- Top 3 most recent orders per customer
ORDER BY 
    c.CustomerID, r.rn;
```

### 🔍 What Happens

* `ROW_NUMBER()` gives a **unique sequence (1, 2, 3, …)** within each `CustomerID` group.
* We then filter where `rn <= 3` → top 3 rows per customer.
* Even if multiple orders share the same `OrderDate`, they still get unique ranks.

---

## ✅ Example 2: Using `RANK()`

```sql
WITH RankedOrders AS (
    SELECT 
        o.OrderID,
        o.CustomerID,
        o.OrderDate,
        o.Amount,
        RANK() OVER (
            PARTITION BY o.CustomerID
            ORDER BY o.OrderDate DESC
        ) AS rnk
    FROM Orders o
)
SELECT 
    c.CustomerID,
    c.CustomerName,
    r.OrderID,
    r.OrderDate,
    r.Amount,
    r.rnk
FROM 
    Customers c
LEFT JOIN RankedOrders r
    ON c.CustomerID = r.CustomerID
   AND r.rnk <= 3   -- Include all tied orders in the top 3 ranks
ORDER BY 
    c.CustomerID, r.rnk, r.OrderID;
```

### 🔍 Key Difference

* `RANK()` assigns the **same rank** to rows with **tied `OrderDate` values**.
* If two orders share the same date, they get the same rank.
* The next rank **skips numbers** (e.g., ranks 1, 1, 3).

So, with `RANK()`:

* You might get **more than 3 rows** if multiple orders are tied for the 3rd position.

---

## 📊 Example Comparison

| Function       | Orders (same date) | Rank Sequence | Total Rows for Top 3       |
| -------------- | ------------------ | ------------- | -------------------------- |
| `ROW_NUMBER()` | Unique per row     | 1, 2, 3, 4    | Always exactly 3 rows      |
| `RANK()`       | Shared for ties    | 1, 1, 3, 4    | Could be > 3 if ties exist |

---

## ✅ Example Output

| CustomerID | CustomerName | OrderID | OrderDate  | Amount | Rank/RowNum       |
| ---------- | ------------ | ------- | ---------- | ------ | ----------------- |
| 1          | Alice        | 101     | 2024-09-10 | 250.00 | 1                 |
| 1          | Alice        | 102     | 2024-08-30 | 180.00 | 2                 |
| 1          | Alice        | 103     | 2024-07-01 | 120.00 | 3                 |
| 2          | Bob          | 201     | 2024-09-10 | 500.00 | 1                 |
| 2          | Bob          | 202     | 2024-09-10 | 500.00 | 1 (if using RANK) |

---

### ⚡ Summary

| Use Case                                                         | Recommended Function | Why                               |
| ---------------------------------------------------------------- | -------------------- | --------------------------------- |
| Always limit to *exactly N* rows per group                       | **`ROW_NUMBER()`**   | Ensures unique sequential numbers |
| Want to include *ties* (e.g., same order date, same score, etc.) | **`RANK()`**         | Keeps logical grouping of ties    |

---

Would you like me to show how to *pivot* those top 3 orders into columns (e.g., `LatestOrder1`, `LatestOrder2`, `LatestOrder3`) for each customer next?

Excellent 😎 — pivoting the **Top 3 latest orders per customer** into **columns** (e.g., `LatestOrder1`, `LatestOrder2`, `LatestOrder3`) is a common reporting pattern.

We’ll do this step-by-step, starting from the ranked query and then pivoting the results.

---

## 🧩 Tables (same setup)

```sql
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    CustomerName VARCHAR(100)
);

CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT,
    OrderDate DATE,
    Amount DECIMAL(10, 2)
);
```

---

## ✅ Step 1: Rank Orders by Recency (Using `ROW_NUMBER()`)

```sql
WITH RankedOrders AS (
    SELECT 
        o.OrderID,
        o.CustomerID,
        o.OrderDate,
        o.Amount,
        ROW_NUMBER() OVER (
            PARTITION BY o.CustomerID
            ORDER BY o.OrderDate DESC
        ) AS rn
    FROM Orders o
)
```

---

## ✅ Step 2: Pivot the Top 3 Orders into Columns

We’ll turn rows with `rn = 1, 2, 3` into separate columns for each customer.

```sql
, PivotedOrders AS (
    SELECT
        CustomerID,
        MAX(CASE WHEN rn = 1 THEN OrderID END) AS LatestOrder1_ID,
        MAX(CASE WHEN rn = 1 THEN OrderDate END) AS LatestOrder1_Date,
        MAX(CASE WHEN rn = 1 THEN Amount END) AS LatestOrder1_Amount,

        MAX(CASE WHEN rn = 2 THEN OrderID END) AS LatestOrder2_ID,
        MAX(CASE WHEN rn = 2 THEN OrderDate END) AS LatestOrder2_Date,
        MAX(CASE WHEN rn = 2 THEN Amount END) AS LatestOrder2_Amount,

        MAX(CASE WHEN rn = 3 THEN OrderID END) AS LatestOrder3_ID,
        MAX(CASE WHEN rn = 3 THEN OrderDate END) AS LatestOrder3_Date,
        MAX(CASE WHEN rn = 3 THEN Amount END) AS LatestOrder3_Amount
    FROM RankedOrders
    GROUP BY CustomerID
)
SELECT 
    c.CustomerID,
    c.CustomerName,
    p.LatestOrder1_ID, p.LatestOrder1_Date, p.LatestOrder1_Amount,
    p.LatestOrder2_ID, p.LatestOrder2_Date, p.LatestOrder2_Amount,
    p.LatestOrder3_ID, p.LatestOrder3_Date, p.LatestOrder3_Amount
FROM 
    Customers c
LEFT JOIN PivotedOrders p ON c.CustomerID = p.CustomerID
ORDER BY c.CustomerID;
```

---

## 🧾 Example Output

| CustomerID | CustomerName | LatestOrder1_ID | LatestOrder1_Date | LatestOrder1_Amount | LatestOrder2_ID | LatestOrder2_Date | LatestOrder2_Amount | LatestOrder3_ID | LatestOrder3_Date | LatestOrder3_Amount |
| ---------- | ------------ | --------------- | ----------------- | ------------------- | --------------- | ----------------- | ------------------- | --------------- | ----------------- | ------------------- |
| 1          | Alice        | 101             | 2024-09-10        | 250.00              | 102             | 2024-08-15        | 180.00              | 103             | 2024-07-01        | 120.00              |
| 2          | Bob          | 201             | 2024-09-01        | 500.00              | NULL            | NULL              | NULL                | NULL            | NULL              | NULL                |
| 3          | Charlie      | NULL            | NULL              | NULL                | NULL            | NULL              | NULL                | NULL            | NULL              | NULL                |

---

### ⚙️ Why This Works

* `ROW_NUMBER()` gives a rank per customer.
* We **pivot manually** using `CASE WHEN rn = N` expressions.
* `MAX()` collapses rows so each customer appears once.

---

### 🧠 Alternatives

1. You could use the **`PIVOT` operator** directly, but the `CASE` method is:

   * More flexible
   * Works in all SQL Server versions
   * Easier to customize (e.g., adding amounts, dates, etc.)

2. If you wanted to handle **ties**, you could replace `ROW_NUMBER()` with `RANK()`, though pivoting gets trickier since `RANK()` can produce duplicate ranks.

---

Would you like me to extend this example to show **Top 3 orders per customer by amount** instead of date next?



Would you like an example with a table-valued function or using `OUTER APPLY` as well?
