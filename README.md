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



Haan, **SQL Injection** tabhi hoti hai jab **user input directly SQL query mein use hota hai bina sanitization aur validation ke**. Yeh ek **critical vulnerability** hai jo attacker ko database manipulate karne, sensitive data churaane, ya even pura system compromise karne ka mauka deti hai.  

## 🔴 **Sabse Dangerous Cases of SQL Injection:**  
Yeh kuch sabse **dangerous aur commonly exploited areas** hain jahan SQLi vulnerability mil sakti hai:  

### **1️⃣ Login Pages (Authentication Bypass)**  
- Aksar **username-password** authentication mein SQLi hoti hai.  
- Example vulnerable query:  
  ```sql
  SELECT * FROM users WHERE username = '$user' AND password = '$pass';
  ```
- **Attack Payload:**  
  ```sql
  ' OR '1'='1' -- 
  ```
  🔥 **Effect:** Attacker **bina password ke login** kar sakta hai.  

---

### **2️⃣ URL Parameters (GET Requests)**  
- Jab query string parameters directly SQL queries mein use hote hain.  
- Example:  
  ```
  https://example.com/product.php?id=5
  ```
  ```sql
  SELECT * FROM products WHERE id = $id;
  ```
- **Attack Payload:**  
  ```
  ?id=5 OR 1=1 --
  ```
  🔥 **Effect:** Pura database expose ho sakta hai.  

---

### **3️⃣ Search Boxes & Filters**  
- Jab **user input directly SQL queries mein inject hota hai**.  
- Example:  
  ```sql
  SELECT * FROM products WHERE name LIKE '%$search%';
  ```
- **Attack Payload:**  
  ```
  %' OR '1'='1' -- 
  ```
  🔥 **Effect:** Attacker har product ka data dekh sakta hai, even restricted ones.  

---

### **4️⃣ Form Inputs (Signup, Contact, Feedback)**  
- Jab **form data bina validation ke database mein store hota hai**.  
- Example:  
  ```sql
  INSERT INTO feedback (name, message) VALUES ('$name', '$message');
  ```
- **Attack Payload:**  
  ```
  '); DROP TABLE feedback; --
  ```
  🔥 **Effect:** **Entire table delete ho sakti hai!**  

---

### **5️⃣ Cookies-based SQL Injection**  
- Jab **cookies bina validation ke SQL query mein use hoti hain**.  
- Example:  
  ```sql
  SELECT * FROM users WHERE session_id = '" . $_COOKIE['session_id'] . "'";
  ```
- **Attack Payload:**  
  ```
  ' OR '1'='1
  ```
  🔥 **Effect:** Attacker kisi bhi user ka session hijack kar sakta hai.  

---

### **6️⃣ HTTP Headers (User-Agent, Referer, X-Forwarded-For, etc.)**  
- **Log files ya analytics** ke liye headers bina sanitization ke store kiye gaye toh SQLi ho sakta hai.  
- Example:  
  ```sql
  INSERT INTO logs (ip, user_agent) VALUES ('" . $_SERVER['REMOTE_ADDR'] . "', '" . $_SERVER['HTTP_USER_AGENT'] . "');
  ```
- **Attack Payload:**  
  ```
  Mozilla/5.0' OR '1'='1' --
  ```
  🔥 **Effect:** Log table manipulate ho sakti hai, ya attacker **database exfiltration** kar sakta hai.  

---

### **7️⃣ Blind SQL Injection (Boolean & Time-based Attacks)**  
- Jab **SQL errors nahi dikhti** par database manipulate ho sakta hai.  
- **Boolean-Based Payload:**  
  ```sql
  ?id=5 AND 1=1 -- (Valid Response)
  ?id=5 AND 1=2 -- (Different Response)
  ```
- **Time-Based Payload:**  
  ```sql
  ?id=5; SLEEP(10) --
  ```
  🔥 **Effect:** Attacker SQL query execution ka **response time analyze** karke database structure samajh sakta hai.  

---

### **8️⃣ Stored SQL Injection (Persistent Attacks)**  
- Jab **malicious SQL query kisi database field mein store ho jaaye** aur later execute ho.  
- Example:  
  ```sql
  INSERT INTO comments (username, comment) VALUES ('$user', '$comment');
  ```
- **Attack Payload:**  
  ```
  '); DROP TABLE users; --
  ```
  🔥 **Effect:** Jab bhi admin ya koi user **comments view karega, toh SQL query execute ho jayegi**, aur **users table delete ho sakta hai!**  

---

### **9️⃣ Second Order SQL Injection**  
- Jab **malicious input pehle store hota hai aur baad mein execute hota hai**.  
- Example:  
  1. Attacker **signup form** mein malicious input daalta hai:  
     ```
     admin' --
     ```
  2. Jab admin panel user ko retrieve karega:  
     ```sql
     SELECT * FROM users WHERE username = '$stored_username';
     ```
  3. SQL query **modify ho jayegi** aur SQLi execute ho sakta hai.  
  🔥 **Effect:** Admin ya kisi bhi user ka account hijack ho sakta hai.  

---

### **🔒 How to Prevent SQL Injection?**  
✅ **Prepared Statements & Parameterized Queries** (Use `mysqli_prepare()` ya `PDO`)  
✅ **Input Validation & Escaping** (`htmlspecialchars()`, `mysqli_real_escape_string()`)  
✅ **Use Web Application Firewalls (WAFs)**  
✅ **Least Privilege Principle (Restrict Database Permissions)**  
✅ **Error Handling & Logging (Don't Display SQL Errors to Users!)**  

---

# SQLMAP TOOL
Overwiew

It looks like you're outlining a structure related to web application security testing, potentially focusing on Burp Suite and SQL injection, with different testing techniques and parameters. I'll go through each of the terms you've listed and explain them briefly.

### Overview
This could be the heading or topic for your web application testing setup, specifically aimed at identifying security flaws like SQL injection, as well as managing configuration settings.

---

### Techniques
These refer to the strategies used for security testing. In the case of SQL injection, Burp Suite offers a variety of ways to test applications for vulnerabilities.

---

### Crawl
Crawling is the process of scanning a web application to map out all its pages and endpoints. It helps the tester identify where to target in terms of testing for vulnerabilities, such as SQL injection.

---

### Enumeration
Enumeration refers to the process of identifying the specific details of an application, such as database names, tables, columns, and more. This is often the next step after identifying a potential injection point.

---

### Batch
Batch processing in the context of Burp Suite could mean running multiple tests or attacks on various parameters or inputs automatically without needing manual intervention for each one.

---

### Risk
Risk is a measure of the potential impact of a vulnerability. The risk level for an SQL injection attack, for example, can be high if an attacker can execute arbitrary queries that expose sensitive data.

---

### Level
Level typically refers to the intensity of the scan or testing being performed. For Burp Suite, this could refer to scan or attack level (such as "Low", "Medium", or "High") when looking for vulnerabilities like SQL injections.

---

### Threads
Threads refer to the number of simultaneous connections or requests that Burp Suite will make during testing. Increasing the number of threads can speed up the process, but it can also impact the server and cause false positives or overloads.

---

### Verbosity
Verbosity controls how much detail is provided in the output of the scan. For example, you can adjust Burp Suite to show only essential information or a more detailed breakdown of each test being performed.

---

### Proxy
Burp Suite's proxy allows you to intercept, modify, and inspect HTTP/S requests and responses between your browser and the target application. It's essential for testing and manipulating web traffic, especially for vulnerability testing like SQL injection.

---

### SQL Injection Via Burp Suite
SQL injection testing is one of the most common security tests. Burp Suite can be used to manipulate user inputs to check if SQL injection vulnerabilities are present. It can automatically identify weak spots by sending malicious payloads to the server to see if it executes unintended SQL queries.

---

### Parameters and Options

- **u (URL)**: This could be the URL or target where the Burp Suite is targeting.
  
- **forms**: Specifies testing of web forms, like login or search forms, which are common places for SQL injection vulnerabilities.

- **--data**: This parameter specifies the data (usually POST data) that will be sent during the request. It's useful for injecting payloads into form submissions.

- **--headers**: Custom headers can be set here. Headers can include authentication tokens or other key information needed for testing.

- **--user-agent**: The User-Agent header identifies the client software making the request. In penetration testing, it might be changed to mimic different browsers or tools.

- **--cookie**: This specifies cookies to be sent with requests. It’s useful when testing sessions and checking for vulnerabilities like session fixation or hijacking.

- **--flush-session**: This forces Burp Suite to clear the current session state, which is useful if you're testing without persistent cookies or need a fresh session for each request.

- **--output-dir**: Specifies the directory to store the results of your scan or attacks.

- **--tamper**: Tampering is the modification of payloads to evade detection by web application firewalls (WAFs) or intrusion detection systems (IDS). Burp Suite allows the use of tamper scripts for this purpose.

---
If you're looking to represent the list of terms you mentioned in the form of a table for GitHub (e.g., in a Markdown file like `README.md`), here’s how you can structure it using Markdown tables.

### Example of a Markdown Table for GitHub

```markdown
| **Technique**               | **Description**                                                                                          |
|-----------------------------|----------------------------------------------------------------------------------------------------------|
| **Crawl**                   | Scan the web application to discover all its endpoints and content.                                      |
| **Enumeration**             | Identifying specific details of an application (e.g., databases, tables, columns).                      |
| **Batch**                   | Running tests or attacks on multiple inputs/parameters automatically in a batch process.                |
| **Risk**                    | Assessing the potential impact or severity of a vulnerability (e.g., High, Medium, Low).                 |
| **Level**                   | The intensity of a scan or test, e.g., Low, Medium, High.                                                |
| **Threads**                 | The number of simultaneous requests that are sent during testing.                                        |
| **Verbosity**               | The amount of detail provided in the output (e.g., minimal or detailed logging).                          |
| **Proxy**                   | Intercepting and modifying HTTP/S traffic between your browser and the target application.               |
| **SQL Injection via Burp**  | Using Burp Suite to identify and exploit SQL injection vulnerabilities.                                  |
| **u**                       | Target URL for testing.                                                                                 |
| **forms**                   | Specify form-based attack vectors (e.g., login forms, search forms) for SQL injection testing.           |
| **--data**                  | Specify the data that will be sent in the request body (typically used for POST requests).               |
| **--headers**               | Custom HTTP headers to send with the request, such as authentication tokens or other key info.           |
| **--user-agent**            | Custom User-Agent string to simulate different browsers or tools.                                        |
| **--cookie**                | Set cookies to simulate authenticated sessions or other relevant session states.                        |
| **--flush-session**         | Clear the current session in Burp Suite, often used for testing without persistent cookies.              |
| **--output-dir**            | Directory to save the results of your scan or attacks.                                                   |
| **--tamper**                | Use tampering scripts to modify payloads and bypass security measures like Web Application Firewalls (WAF).|
```

---

### How It Will Render:

| **Technique**               | **Description**                                                                                          |
|-----------------------------|----------------------------------------------------------------------------------------------------------|
| **Crawl**                   | Scan the web application to discover all its endpoints and content.                                      |
| **Enumeration**             | Identifying specific details of an application (e.g., databases, tables, columns).                      |
| **Batch**                   | Running tests or attacks on multiple inputs/parameters automatically in a batch process.                |
| **Risk**                    | Assessing the potential impact or severity of a vulnerability (e.g., High, Medium, Low).                 |
| **Level**                   | The intensity of a scan or test, e.g., Low, Medium, High.                                                |
| **Threads**                 | The number of simultaneous requests that are sent during testing.                                        |
| **Verbosity**               | The amount of detail provided in the output (e.g., minimal or detailed logging).                          |
| **Proxy**                   | Intercepting and modifying HTTP/S traffic between your browser and the target application.               |
| **SQL Injection via Burp**  | Using Burp Suite to identify and exploit SQL injection vulnerabilities.                                  |
| **u**                       | Target URL for testing.                                                                                 |
| **forms**                   | Specify form-based attack vectors (e.g., login forms, search forms) for SQL injection testing.           |
| **--data**                  | Specify the data that will be sent in the request body (typically used for POST requests).               |
| **--headers**               | Custom HTTP headers to send with the request, such as authentication tokens or other key info.           |
| **--user-agent**            | Custom User-Agent string to simulate different browsers or tools.                                        |
| **--cookie**                | Set cookies to simulate authenticated sessions or other relevant session states.                        |
| **--flush-session**         | Clear the current session in Burp Suite, often used for testing without persistent cookies.              |
| **--output-dir**            | Directory to save the results of your scan or attacks.                                                   |
| **--tamper**                | Use tampering scripts to modify payloads and bypass security measures like Web Application Firewalls (WAF).|

---

This table format is highly readable and provides an organized way to explain different parameters, techniques, and settings related to Burp Suite or web application security testing. You can paste this into your GitHub repository's `README.md` file, and it will render neatly for others to understand easily.

Let me know if you'd like to further customize the table or add more rows!




