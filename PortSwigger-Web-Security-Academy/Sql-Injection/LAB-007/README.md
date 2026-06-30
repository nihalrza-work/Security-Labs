# LAB-007: SQL Injection - Determining the Number of Columns Returned by the Query

## Objective

To identify and exploit a SQL Injection vulnerability to determine the exact number of columns being returned by the application's backend query, establishing the groundwork for a subsequent UNION-based data exfiltration attack.

---

## Vulnerability Type

SQL Injection (UNION-based)

---

## Target

Category filter parameter vulnerable to SQL Injection and capable of reflecting query results in the application's response.

---

## Context

The application concatenates user input directly into an SQL query without proper sanitization or parameterization. By injecting a `UNION SELECT` statement, an attacker can append rows to the application's result set. However, a `UNION` attack strictly requires the injected query to return the exact same number of columns as the original query.

---

## Approach

### 1. Observation
- Identified that the category filter parameter was vulnerable to SQL Injection.
- Confirmed that successful database query results were reflected inside the product table display on the webpage.
- Determined that any structural mismatch in columns triggered an application error, allowing for behavioral feedback during structural discovery.

### 2. Column Enumeration
- Executed a systematic layout check by progressively injecting URL-encoded null values into a `UNION SELECT` statement.
- Used `NULL` placeholders because they are automatically convertible to any standard data type, preventing data-type mismatch errors during structural discovery.
- Discovered that injecting an incorrect number of `NULL` constants resulted in an application error, but shifting to exactly three `NULL` values (`NULL%2C+NULL%2C+NULL`) caused the page to load successfully and show the product list. This confirmed the backend query processes exactly three (3) columns.

### 3. Database Isolation & Elimination (Fingerprinting)
To safely identify the backend engine without causing a fatal syntax error, a structured elimination methodology was executed based on behavior analysis:

* **Oracle Elimination:** The payload did not require a `FROM dual` clause to execute, immediately eliminating Oracle from the pool of possibilities.
* **PostgreSQL Confirmation via Concatenation:** To isolate the exact backend database, string manipulation capabilities were analyzed. Testing verified that the backend successfully evaluated string concatenation using both the pipeline operator (`||`) and standard `CONCAT('a', 'b')` syntax. Combined with the absence of specific MySQL or MSSQL query quirks, the backend engine was definitively fingerprint-confirmed as PostgreSQL.

### 4. Exploitation
- Crafted the final payload stacking a three-column URL-encoded `UNION SELECT` filled with compatible `NULL` markers directly into the vulnerable category filter.
- Verified that the application successfully merged the injected rows and rendered the page framework smoothly.

---

## Payloads Used
`'+UNION+SELECT+NULL%2C+NULL%2C+NULL`

---

## Result

The application successfully executed the payload and returned a standard `200 OK` HTTP response, rendering the product listing framework cleanly while proving a 3-column architecture without disrupting the backend application logic.

---

## Impact

- Direct structural disclosure of the backend database query layout.
- Establishes the precise column count required to weaponize subsequent `UNION` payloads.
- Lays the groundwork for targeted data extraction, including dumping database tables, schemas, and sensitive user credentials.

---

## Key Insight

When testing for UNION-based SQL injection, utilizing `NULL` constants is the most reliable strategy for column enumeration. Because `NULL` satisfies any data type constraint (strings, integers, booleans, etc.) across different database engines, it isolates the structural variable (column count) completely from data type mismatches. 

Once the application returns a valid response instead of a server error, the structural barrier is broken, allowing the operator to map out text-supported columns next.

---

## Mitigation

- **Prepared Statements:** Implement strongly typed, parameterized queries to cleanly separate user data from command logic.
- **Input Filtering:** Apply strict server-side validation against incoming parameters using an allow-list model.
- **Defensive Error Handling:** Ensure all application exceptions are handled gracefully, preventing technical metadata or verbose server errors from leaking structural clues to the browser.
- **Least Privilege:** Enforce the principle of least privilege on web-facing database connection strings to minimize the impact of data exposure.

---

## Evidence

- Screenshot 01 : Successful rendering of the interface with the `'+UNION+SELECT+NULL%2C+NULL%2C+NULL` payload executed, showing the application status changed from error states to a solved/active state.

**Lab Link:** https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns
