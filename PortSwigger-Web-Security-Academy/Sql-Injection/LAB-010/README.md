# LAB-010: SQL Injection UNION Attack - Retrieving Multiple Values in a Single Column

## Objective

To identify and exploit a SQL Injection vulnerability on a PostgreSQL backend where the target query only returns a single column capable of holding text, necessitating the use of string concatenation (`||`) to extract multiple data fields (usernames and passwords) simultaneously within that single reflection point.

---

## Vulnerability Type

SQL Injection (UNION-based)

---

## Target

Category filter parameter vulnerable to SQL Injection and capable of reflecting concatenated data rows into a single text-compatible layout position.

---

## Context

The web application directly joins user-supplied text parameters into structural backend SQL paths. However, the constraints of this environment differ from previous exercises: the underlying query architecture processes fewer text fields. When an application's layout limits data extraction to only one text-friendly column, multi-field database data (like credentials) must be structurally fused together into a single string stream prior to evaluation by the database engine.

---

## Approach

### 1. Observation
- Identified that the category filter parameter was vulnerable to SQL Injection.
- Confirmed that successful database query results were reflected inside the product listing display on the webpage.

### 2. Column Enumeration via ORDER BY
- Executed a structural integrity check using the `ORDER BY` clause to determine total columns.
- Incrementing the index (`ORDER BY 1`, `ORDER BY 2`) loaded normally, but attempting to sort by a third index (`ORDER BY 3`) triggered a `500 Internal Server Error`. This confirmed the backend query returns exactly two (2) columns.

### 3. Database Isolation & Elimination (Fingerprinting)
To safely identify the backend engine without causing a fatal syntax error, a structured elimination methodology was executed based on behavior analysis:

* **Oracle Elimination:** The payload did not require a `FROM dual` clause to execute, immediately eliminating Oracle from the pool of possibilities.
* **PostgreSQL Confirmation via Concatenation:** To isolate the exact backend database, string manipulation capabilities were analyzed. Testing verified that the backend successfully evaluated string concatenation using both the pipeline operator (`||`) and standard `CONCAT('a', 'b')` syntax. Combined with the absence of specific MySQL or MSSQL query quirks, the backend engine was definitively fingerprint-confirmed as PostgreSQL.

### 4. Schema Mapping & Target Identification
- **Data Type Mapping:** Having established a 2-column baseline, individual column positions were evaluated. Injecting a text literal into the second position successfully rendered onto the interface, revealing that only one column (Column 2) was capable of handling string parameters safely without causing database type-mismatch faults.
- **Table Enumeration:** Queried the standard metadata views via `'+UNION+SELECT+NULL,+table_name+FROM+information_schema.tables+--` to dump existing tables. Located a sensitive storage source named `users`.
- **Column Enumeration:** Isolated target field structures inside the database view by querying `information_schema.columns`. Evaluated properties associated with the `users` source, revealing the target column names `username` and `password`.

### 5. Exploitation (Data Merging & Authentication Bypass)
- Because only Column 2 could display string data, a simple dual-column `SELECT username, password` was impossible as it would conflict with Column 1's non-string data constraints.
- To bypass this bottleneck, PostgreSQL's native string joining operator (`||`) was utilized to blend multiple properties along with an explicit separator string (`'@69@'`).
- The payload injected was: `'+UNION+SELECT+NULL,+username||'%4069%40'||password+FROM+users+--`.
- The database successfully compacted the administrative dataset into a single text block per row and displayed them sequentially on screen.
- Extracted credentials for the principal user: `administrator` : `1bjjgtfly6r0xd64chus`.
- Navigated to the `/login` portal, supplied the exfiltrated properties, and successfully authenticated into the account management system.

---

## Payloads Used
`'+UNION+SELECT+NULL,+username||'%4069%40'||password+FROM+users+--`

---

## Result

The application successfully executed the multi-value concatenation payload, bypassing data-type constraints and streaming plain-text password tokens into the single text-capable column. Applying the parsed properties allowed the authentication logic to evaluate successfully, logging into the application as the system administrator and resolving the lab.

---

## Impact

- Full administrative account takeover and privilege escalation.
- Mass leakage of sensitive multi-column datasets through restrictive, single-reflection application points.
- Complete bypass of strong application formatting rules and database row constraints via data type encapsulation.

---

## Key Insight

A common developer defense is to limit the number of data fields displayed on a front-end view to minimize the exposure surface of a `UNION` attack. However, data type constraints only restrict individual structural cells, not the content packed inside them. 

By leveraging native database string functions like the pipeline operator (`||`), an operator can compress an entire database row containing dozens of columns into a single string object. Using an explicit, non-standard separator token (like `@69@`) ensures the combined string stream can be cleanly parsed back into distinct variables on the operator's end, rendering multi-column field limits ineffective.

---

## Mitigation

- **Prepared Statements:** Implement strongly typed, parameterized queries to cleanly separate user data from command logic.
- **Input Filtering:** Apply strict server-side validation against incoming parameters using an allow-list model.
- **Defensive Error Handling:** Ensure all application exceptions are handled gracefully, preventing technical metadata or verbose server errors (`500 Internal Server Error`) from leaking structural clues to the browser.
- **Least Privilege:** Enforce the principle of least privilege on web-facing database connection strings to minimize the impact of data exposure.

---

## Evidence

- Reflective extraction showing the table maps, column constraints, and the resulting concatenated administrator credential set (`administrator@69@1bjjgtfly6r0xd64chus`) outputting cleanly to the user interface framework.
- Referenced files: `01-found-single-column-containing-string.png`, `02-displayed-tables.png`, `03-target-table.png`, `04-target-columns.png`, `05-admin-password-reveal.png`, `06-admin-login.png`

**Lab Link:** https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-multiple-values-in-single-column
