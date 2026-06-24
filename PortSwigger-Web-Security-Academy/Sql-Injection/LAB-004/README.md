# LAB-004: SQL Injection - Querying Database Type and Version (MySQL/Microsoft)

## Objective

To identify and exploit a SQL Injection vulnerability that allows disclosure of the backend database type and version information by leveraging a UNION-based query on a MySQL backend.

---

## Vulnerability Type

SQL Injection (UNION-based)

---

## Target

Category filter parameter vulnerable to SQL Injection and capable of reflecting query results in the application's response.

---

## Context

The application concatenates user input directly into an SQL query without proper sanitization or parameterization. Since the query results are directly reflected in the UI response, a UNION-based SQL Injection can be used to retrieve system information and global variables from a MySQL database engine.

---

## Approach

### 1. Observation
- Identified that the category filter parameter was vulnerable to SQL Injection.
- Confirmed that database query results were reflected inside the product table display on the webpage.
- Proved that the application returns responses based on logical execution, confirming a UNION-based approach was feasible.

### 2. Column Enumeration
- Determined that the original query returned exactly two columns.
- Used a systematic layout check to find a column capable of displaying text, using `NULL` placeholders to maintain data type compatibility across both columns.

### 3. Database Isolation & Elimination (Fingerprinting)
To safely identify the backend engine without causing a fatal syntax error, a structured elimination methodology was executed:

* **Oracle Elimination via `information_schema`:** An `EXISTS` check was performed against standard metadata tables using `Pets' AND EXISTS (SELECT * FROM information_schema.tables) --+`. The application loaded the product page normally. Because Oracle utilizes its own system views (like `all_tables`) and entirely lacks `information_schema`, this successfully confirmed the environment was a non-Oracle database (MySQL, PostgreSQL, or MS SQL).
* **PostgreSQL vs. MySQL Isolation via Concatenation:** To narrow down the remaining engines, distinct native string operators were tested:
    * *The Pipeline Operator (`||`):* Testing PostgreSQL's native string joining syntax (`'a'||'b' = 'ab'`) failed to yield positive results or threw a mismatch.
    * *The Space Quirk:* MySQL automatically concatenates string literals placed next to each other separated only by a space. Testing the payload `Pets' AND 'ab' = 'a' 'b' --+` resulted in the normal rendering of products. Because this explicit structural behavior is unique to MySQL, the backend database engine was definitively fingerprint-confirmed.

### 4. Exploitation
- Queried MySQL’s global system variable `@@version` to retrieve the database configuration string.
- Constructed the query without a `FROM` clause, utilizing MySQL's structural capacity to process standalone selects for global variables.

---

## Payloads Used
' UNION SELECT @@version, NULL --

---

## Result

The application successfully executed the payload and returned the specific MySQL version string into the application's product listing framework, cleanly disclosing the technology and exact version running on the server.

---

## Impact

- Direct database fingerprinting and underlying technology disclosure.
- Exposure of specific patch-level and minor version information.
- Enables attackers to reference known, version-specific MySQL vulnerabilities and exploits.
- Drastically reduces attacker uncertainty, laying groundwork for localized reconnaissance and further exploitation.

---

## Key Insight

Unlike databases like Oracle which strictly mandate a dummy table reference (`FROM dual`), MySQL natively permits standalone `SELECT` statements when querying global variables, system functions, or literal constants. 

When a web application's rendering engine loops through database query columns and dumps them to the HTML layout, an attacker can seamlessly stack their own system metadata using `UNION` without needing to map or locate an active table source.

---

## Mitigation

- **Prepared Statements:** Implement strongly typed, parameterized queries to cleanly separate user data from command logic.
- **Input Filtering:** Apply strict server-side validation against incoming parameters using an allow-list model.
- **Defensive Error Handling:** Ensure all application exceptions are handled gracefully, preventing technical metadata or verbose errors from leaking to the browser.
- **Least Privilege:** Enforce the principle of least privilege on web-facing database connection strings to minimize the impact of data exposure.

---

## Evidence
- Screenshot 01 : Lab Solved
- Screenshot 02 : Showing the MySQL version banner printed cleanly on the bottom of the product page.

**Lab Link:** https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-mysql-microsoft
