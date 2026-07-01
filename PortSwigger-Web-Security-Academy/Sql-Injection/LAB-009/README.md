# LAB-009: SQL Injection UNION Attack - Retrieving Data from Other Tables

## Objective

To identify and exploit a SQL Injection vulnerability on a PostgreSQL backend to extract cryptographic or plain-text user credentials from a non-public application table, ultimately gaining unauthorized administrative access.

---

## Vulnerability Type

SQL Injection (UNION-based)

---

## Target

Category filter parameter vulnerable to SQL Injection and capable of reflecting multi-column query results from alternative database tables into the application interface.

---

## Context

The web application directly interpolates unvalidated user inputs into structural database query execution paths. Since the structural constraints (3 columns total, with the 1st and 2nd columns capable of mapping text fields safely) have been enumerated, an attacker can substitute the dummy elements of a `UNION SELECT` statement with specific table properties (`username`, `password`) to bypass structural barriers and display target schema contents directly to the frontend interface.

---

## Approach

### 1. Observation
- Identified that the category filter parameter was vulnerable to SQL Injection.
- Confirmed that successful database query results were reflected inside the product table display on the webpage.

### 2. Column Enumeration via ORDER BY
- Executed a systematic layout check using the `ORDER BY` clause to determine total columns.
- Incrementing the index (`ORDER BY 1`, `ORDER BY 2`, `ORDER BY 3`) loaded normally, but attempting to sort by the fourth index (`ORDER BY 4`) triggered a `500 Internal Server Error`. This confirmed the backend query returns exactly three (3) columns.

### 3. Database Isolation & Elimination (Fingerprinting)
To safely identify the backend engine without causing a fatal syntax error, a structured elimination methodology was executed based on behavior analysis:

* **Oracle Elimination:** The payload did not require a `FROM dual` clause to execute, immediately eliminating Oracle from the pool of possibilities.
* **PostgreSQL Confirmation via Concatenation:** To isolate the exact backend database, string manipulation capabilities were analyzed. Testing verified that the backend successfully evaluated string concatenation using both the pipeline operator (`||`) and standard `CONCAT('a', 'b')` syntax. Combined with the absence of specific MySQL or MSSQL query quirks, the backend engine was definitively fingerprint-confirmed as PostgreSQL.

### 4. Schema Mapping & Target Identification
- **Table Enumeration:** Queried the standard metadata views via `'+UNION+SELECT+table_name,+NULL,+NULL+FROM+information_schema.tables+--` to dump existing tables. Located a sensitive custom storage source named `users`.
- **Column Enumeration:** Target column definitions were isolated by querying `information_schema.columns`. Evaluated column names associated with the target source, successfully discovering the field labels `username` and `password`.

### 5. Exploitation (Data Exfiltration & Authentication Bypass)
- Structured a tailored extraction statement mapping the data types directly: `'+UNION+SELECT+username,+password,+NULL+FROM+users+--`.
- The database successfully processed the cross-table join, appending administrative credential strings onto the visible page layout.
- Extracted credentials for the principal user: `administrator` : `r5ni10sywbdbk5ypofpa`.
- Navigated to the `/login` portal, supplied the exfiltrated properties, and successfully authenticated into the account management system.

---

## Payloads Used
`'+UNION+SELECT+username,+password,+NULL+FROM+users+--`

---

## Result

The application successfully executed the multi-table extraction payload and returned the plain-text password tokens for accounts stored in the backend. Applying these credentials allowed the account access flag to evaluate successfully, logging into the application as the system administrator and resolving the lab.

---

## Impact

- Full administrative account takeover and privilege escalation.
- Mass leakage of user account databases, including active passwords or target configurations.
- Complete breach of horizontal and vertical authorization access layers within the application tier.

---

## Key Insight

A UNION-based SQL injection allows an attacker to treat the vulnerable application query as a proxy execution interface to dump external tables. Once the structural barriers—such as column numbers and individual cell types—are determined via systematic testing, the application behaves as an open visualization layer for any table the web database user account has permissions to read. 

By systematically traversing from `information_schema.tables` down to specific column rows, the internal structural architecture can be weaponized to pull operational account information without breaking core runtime execution.

---

## Mitigation

- **Prepared Statements:** Implement strongly typed, parameterized queries to cleanly separate user data from command logic.
- **Input Filtering:** Apply strict server-side validation against incoming parameters using an allow-list model.
- **Defensive Error Handling:** Ensure all application exceptions are handled gracefully, preventing technical metadata or verbose server errors (`500 Internal Server Error`) from leaking structural clues to the browser.
- **Least Privilege:** Enforce the principle of least privilege on web-facing database connection strings to minimize the impact of data exposure.

---

## Evidence

- Reflective extraction showing the table maps, column types, and the resulting plain-text administrator credential set (`administrator` / `r5ni10sywbdbk5ypofpa`) outputting cleanly to the user interface framework.
- Referenced files: `01-listed-all-table-names.png`, `02-target-table.png`, `03-column-names-found.png`, `04-admin-credentials-found.png`, `05-logged-in-as-login.png`

**Lab Link:** https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-data-from-other-tables
