# SQL_Injection
SQL Injection (SQLi) vulnerability commonly milta hai un web applications mein jo user input ko directly SQL queries ke sath execute karte hain bina proper validation aur sanitization ke. Yeh vulnerabilities aksar following jagahon par payi ja sakti hain:  

### 1. **Login Pages (Authentication Bypass)**
   - Jab username aur password directly SQL query mein inject kiya jata hai, bina input validation ke.  
   - Example:  
     ```sql
     SELECT * FROM users WHERE username = '$user' AND password = '$pass';
     ```
     Agar input `admin' --` diya jaye, toh authentication bypass ho sakta hai.  

### 2. **Search Bars / Filters**
   - Jab user ke search input ko bina sanitize kiye SQL query mein inject kiya jata hai.  
   - Example:  
     ```sql
     SELECT * FROM products WHERE name LIKE '%$search%';
     ```
     Agar input `"%' OR '1'='1"` diya jaye, toh pura database expose ho sakta hai.  

### 3. **URL Parameters**
   - Agar application URL parameters directly SQL query mein pass kar rahi ho.  
   - Example:  
     ```
     https://example.com/product.php?id=5
     ```
     Query:  
     ```sql
     SELECT * FROM products WHERE id = $id;
     ```
     Agar `id=5 OR 1=1` pass kiya jaye, toh saare products fetch ho sakte hain.  

### 4. **Forms (Feedback, Contact, Registration)**
   - Forms jo user ke input ko directly database mein insert karte hain.  
   - Example:  
     ```sql
     INSERT INTO feedback (name, message) VALUES ('$name', '$message');
     ```
     Agar input `' ); DROP TABLE feedback; --` diya jaye, toh table delete ho sakta hai.  

### 5. **Admin Panels / CMS**
   - Jab admin dashboard par SQL queries execute ki jati hain bina escaping ke.  
   - Example:  
     ```sql
     SELECT * FROM users WHERE role = '$role';
     ```
     Agar input `' OR '1'='1'` diya jaye, toh saare users ka data expose ho sakta hai.  

SQL Injection (SQLi) vulnerabilities aur bhi jagahon par mil sakti hain, khas kar un web applications mein jo user input ko bina validation ya sanitization ke directly SQL queries mein use karti hain. Yahaan kuch additional jagah hain jahan SQLi vulnerability mil sakti hai:  

---

### **6. Cookies-based SQL Injection**  
Agar application user ke session ya authentication ke liye cookies ka use karti hai aur directly SQL query mein include karti hai, toh SQLi ho sakta hai.  

✅ **Example:**  
```php
$query = "SELECT * FROM users WHERE session_id = '" . $_COOKIE['session_id'] . "'";
```  
Agar koi attacker malicious cookie set kare:  
```sql
' OR '1'='1
```
Toh woh kisi bhi user ka data access kar sakta hai.

---

### **7. HTTP Headers (User-Agent, Referer, X-Forwarded-For, etc.)**  
Agar web application incoming HTTP headers ko bina validate kiye SQL queries mein daalti hai, toh SQLi possible hai.  

✅ **Example:**  
```php
$query = "INSERT INTO logs (ip, user_agent) VALUES ('" . $_SERVER['REMOTE_ADDR'] . "', '" . $_SERVER['HTTP_USER_AGENT'] . "')";
```
Agar **User-Agent** modify karke SQL injection payload diya jaye:  
```sql
Mozilla/5.0' OR '1'='1' --
```
Toh attacker logs manipulate kar sakta hai ya even database dump kar sakta hai.

---

### **8. Stored SQL Injection (Persistent SQLi)**  
Agar SQL injection vulnerability kisi aise input field mein ho jo data store karta ho (e.g., comments, reviews, feedback), toh SQLi ka payload future requests mein bhi execute ho sakta hai.

✅ **Example:**  
Agar ek forum post ya comment box bina sanitization ke data store karta hai:  
```php
$query = "INSERT INTO comments (username, comment) VALUES ('$user', '$comment')";
```
Agar koi user comment kare:  
```sql
'); DROP TABLE users; --
```
Toh jab bhi yeh comment fetch hoga, query break ho sakti hai aur table delete ho sakta hai.

---

### **9. Second Order SQL Injection**  
Is case mein, attacker pehli request mein malicious input store karta hai, jo kisi dusri query mein execute hota hai.  

✅ **Example:**  
1. Attacker sign-up form mein apna **username** kuch aisa daalta hai:  
   ```sql
   admin' -- 
   ```
2. Yeh data database mein store ho jata hai.  
3. Jab admin panel user ko retrieve karta hai:  
   ```php
   $query = "SELECT * FROM users WHERE username = '$stored_username'";
   ```
4. Toh attacker ka injected payload execute ho jata hai.

---

### **10. XML Data Injection (SQLi via XML Requests)**  
Agar web application XML-based APIs ka use karti hai aur input validate nahi karti, toh SQLi ho sakta hai.  

✅ **Example:**  
```xml
<user>
    <id>1' OR '1'='1</id>
</user>
```
Agar backend SQL query kuch aisa execute karta hai:  
```php
$query = "SELECT * FROM users WHERE id = '$id'";
```
Toh yeh vulnerability cause kar sakti hai.

---

### **11. NoSQL Injection (MongoDB, Firebase, etc.)**  
SQL ke alawa NoSQL databases (MongoDB, Firebase, CouchDB) bhi injection vulnerabilities se affect ho sakte hain.

✅ **Example (MongoDB):**  
```php
$search = $_GET['search'];
$query = array("name" => $search);
$cursor = $collection->find($query);
```
Agar attacker kuch aisa input kare:  
```json
{ "$ne": null }
```
Toh woh saara data fetch kar sakta hai.

---

### **12. Blind SQL Injection (Boolean & Time-Based)**  
Agar application SQL error messages return nahi karti, tab bhi SQL injection possible hai **Boolean-based** ya **Time-based** techniques se.

✅ **Example:**  
```sql
?id=5 AND 1=1 --  (Valid Response)
?id=5 AND 1=2 --  (Different Response)
```
Ya phir time-based attack:  
```sql
?id=5 AND SLEEP(10) --
```
Agar page response time increase ho jaye, toh SQLi confirm ho sakti hai.

---

### **13. SQL Injection in Backup Files (.sql, .bak, .db, etc.)**  
Agar kisi website ke **database backup files** (e.g., `database.sql`, `backup.bak`) publicly accessible hain, toh attacker unko download karke direct SQL injection ya database enumeration kar sakta hai.

✅ **Mitigation:**  
- Backup files ko publicly accessible directories mein store na karein.  
- `.htaccess` ya firewall rules use karke `.sql`, `.bak`, `.db` files ko block karein.

---

### **14. SQL Injection in Stored Procedures**  
Agar database stored procedures bina input validation ke queries execute karti hain, toh SQLi ho sakta hai.

✅ **Example:**  
```sql
CREATE PROCEDURE GetUser(IN userID VARCHAR(50))
BEGIN
    SET @sql = CONCAT('SELECT * FROM users WHERE id = ', userID);
    PREPARE stmt FROM @sql;
    EXECUTE stmt;
    DEALLOCATE PREPARE stmt;
END;
```
Agar `userID` parameter properly sanitized nahi kiya gaya, toh SQLi possible hai.

---

### **15. SQL Injection in Mobile Apps & APIs**  
Agar mobile apps ya REST APIs insecurely user input ko SQL queries mein inject karti hain, toh SQLi ho sakta hai.

✅ **Example:**  
```json
{
  "username": "admin' --",
  "password": "password123"
}
```
Agar API backend directly SQL query execute kare bina validation ke:  
```php
$query = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";
```
Toh attacker authentication bypass kar sakta hai.

---

## **Conclusion:**
🚨 **SQL Injection har jagah ho sakti hai jahan user input directly SQL query mein use hoti hai bina sanitization ke.**  
🔴 **Sabse dangerous cases:**  
- Login pages  
- Search bars  
- URL parameters  
- API requests  
- HTTP headers & cookies  
- Backup files & stored procedures  

---

# SQLMAP TOOL
Overwiew
---

| **Technique**               | **Description**                                                                                              |
|-----------------------------|--------------------------------------------------------------------------------------------------------------|
| **--crawl**                 | Scans the web application to discover all endpoints and content (e.g., `--crawl 5`).                        |
| **Enumeration**             | Identifies specific details of an application, such as databases, tables, and columns.                     |
| **--batch**                 | Executes tests or attacks on multiple inputs/parameters automatically in a batch process.                   |
| **Risk Assessment**         | Evaluates the potential impact or severity of a vulnerability (e.g., High, Medium, Low).                     |
| **--level**                 | Defines the intensity of a scan or test (e.g., Low, Medium, High). - ![LEVEL](https://github.com/Dharmendrastm/SQL_Injection/blob/main/Level.png)                                          |
| **--threads**               | Specifies the number of simultaneous requests sent during testing.                                          |
| **Verbosity Levels**        | Controls the level of detail in output logs:- ![VERBOSITY](https://github.com/Dharmendrastm/SQL_Injection/blob/main/Verbosity.png)                                                                |                                                    |
| **--proxy**                 | Intercepts and modifies HTTP/S traffic between the browser and the target application.                      |
| **--SQL Injection via Burp** | Uses Burp Suite to identify and exploit SQL injection vulnerabilities.                                      |
| **-u**                      | Specifies the target URL for testing.                                                                      |
| **--forms**                 | Targets form-based attack vectors (e.g., login forms, search fields) for SQL injection testing.            |
| **--data**                  | Defines the data sent in the request body (typically used for POST requests).                              |
| **--headers**               | Adds custom HTTP headers to requests, such as authentication tokens.                                       |
| **--user-agent**            | Sets a custom User-Agent string to simulate different browsers or tools.                                   |
| **--cookie**                | Sets cookies to simulate authenticated sessions or specific session states.                               |
| **--flush-session**         | Clears the current session in Burp Suite, often used for testing without persistent cookies.              |
| **--output-dir**            | Specifies the directory to save scan results or attack logs.                                               |
| **--tamper**                | Uses tampering scripts to modify payloads and bypass security measures, such as Web Application Firewalls (WAF). |


---
**SQLmap** is a powerful open-source tool used for automating the process of detecting and exploiting SQL injection vulnerabilities in web applications. It provides a variety of techniques to exploit different types of SQL injection vulnerabilities. Below are the key techniques in SQLmap, each serving a specific purpose for testing and exploiting SQL injection vulnerabilities.

### Key Techniques of SQLmap

1. **Boolean-based Blind SQL Injection**
   - **Technique**: SQLmap sends a series of queries that return either `True` or `False` based on whether the payload works. This technique doesn't return any data directly but relies on the behavior of the application (e.g., page content, HTTP status codes, response time) to infer the database's response.
   - **Use Case**: When no data is displayed in response, but there are changes in behavior based on SQL injection.
   - **Example**: `id=1 AND 1=1` vs. `id=1 AND 1=2`

2. **Time-based Blind SQL Injection**
   - **Technique**: In a time-based blind SQL injection, SQLmap injects payloads that cause a delay in the application's response if the SQL query is executed successfully. The tool then measures the response time to determine whether the query is vulnerable.
   - **Use Case**: When the application does not give any visible feedback but delays occur when certain inputs are processed.
   - **Example**: `id=1 AND SLEEP(5)` will delay the response by 5 seconds if successful.

3. **Error-based SQL Injection**
   - **Technique**: SQLmap uses error messages generated by the database to gain information about the structure of the database, tables, and columns. This method involves exploiting database errors to gather detailed information about the underlying database.
   - **Use Case**: When the application reveals database error messages in the response that can be used to infer database structure.
   - **Example**: `id=1'` could trigger an error like "You have an error in your SQL syntax."

4. **Union-based SQL Injection**
   - **Technique**: SQLmap uses `UNION` SQL queries to combine the results of multiple queries into a single result. This allows the tool to extract data from other database tables and columns.
   - **Use Case**: When SQL injection allows the combination of results from two or more SELECT queries.
   - **Example**: `id=1 UNION SELECT null, username, password FROM users`

5. **Out-of-Band (OOB) SQL Injection**
   - **Technique**: This technique relies on the database making HTTP or DNS requests to an external server controlled by the attacker. SQLmap listens for these requests to confirm the injection and sometimes to retrieve data.
   - **Use Case**: When the database is configured to make outbound requests (e.g., for error logging or external communication).
   - **Example**: Injecting a payload that triggers a DNS request to a malicious server.

6. **Stacked Queries (Multiple Queries)**
   - **Technique**: SQLmap exploits SQL injection to send multiple queries in one request. This is used when the application allows multiple SQL queries to be executed in a single request (i.e., query chaining).
   - **Use Case**: When the application allows multiple queries to be executed in one request, typically found in less-restricted configurations.
   - **Example**: `id=1; DROP TABLE users;` (The second query executes after the first one).

7. **Stored Procedures**
   - **Technique**: SQLmap identifies and exploits stored procedures (predefined database functions) that may be susceptible to SQL injection. Stored procedures can be used to execute malicious commands or retrieve data.
   - **Use Case**: When stored procedures are used in the application, and they are vulnerable to SQL injection.
   - **Example**: Executing a stored procedure like `sp_configure` to extract information from the database.

8. **Second-Order SQL Injection**
   - **Technique**: This occurs when the payload isn't immediately executed but is stored (e.g., in a database or cache) and executed later. SQLmap can detect and exploit such vulnerabilities by triggering a stored payload.
   - **Use Case**: When an SQL injection payload is not executed directly, but results are triggered later (such as by displaying data stored in a database).
   - **Example**: Injecting a payload through a form that is stored in a session variable or database and executed when viewed later.

9. **Tampering (SQLmap Tamper Scripts)**
   - **Technique**: SQLmap allows the use of **tamper scripts** to modify the injected payloads to bypass filters and WAFs (Web Application Firewalls). Tamper scripts can obfuscate the payload or alter its syntax.
   - **Use Case**: To bypass security measures like WAFs, IDS/IPS systems, or input sanitization filters that may block direct SQL injection attempts.
   - **Example**: Using a tamper script like `charencode` to obfuscate the payload (e.g., converting `1=1` into its encoded form).

10. **Database Fingerprinting**
   - **Technique**: SQLmap can detect the type and version of the underlying database system by using various queries to extract specific database properties.
   - **Use Case**: When the database type/version isn't initially known, and it's necessary to tailor the injection to the correct SQL dialect.
   - **Example**: SQLmap will try queries to determine if the database is MySQL, PostgreSQL, Oracle, etc.

11. **DBMS Detection**
   - **Technique**: SQLmap can detect the Database Management System (DBMS) by testing the behavior of SQL queries. This allows SQLmap to adjust its injection techniques based on the target's DBMS.
   - **Use Case**: To determine which DBMS is in use and tailor attacks accordingly.
   - **Example**: Running tests to determine if the target is using MySQL, Oracle, Microsoft SQL Server, etc.

12. **User-Agent and Referrer Detection**
   - **Technique**: SQLmap can use different headers such as `User-Agent` and `Referrer` to identify and exploit vulnerabilities based on how the server reacts to different inputs.
   - **Use Case**: To mimic different browsers, devices, or referrers to bypass detection and exploit SQL injection vulnerabilities.
   - **Example**: Modifying the `User-Agent` to trick the application into behaving differently during the attack.








