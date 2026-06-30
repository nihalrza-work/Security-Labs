# LAB-008: SQL Injection UNION Attack - Finding a Column Containing Text

## Objective

To identify a specific column within the query result set that can accept and display string/text data, allowing for the extraction of sensitive textual information from a PostgreSQL backend.

---

## Vulnerability Type

SQL Injection (UNION-based)

---

## Target

Category filter parameter vulnerable to SQL Injection and capable of reflecting query results in the application's response.

---

## Context

The application concatenates user input directly into an SQL query without proper sanitization or parameterization. After establishing that the database query processes three columns, a UNION attack requires matching the exact data types of each column. Testing columns sequentially with string values determines which column can safely reflect data without causing data-type mismatch errors in the database backend.

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

### 4. Exploitation (Data Type Mapping)
- Having confirmed 3 columns, individual columns were systematically tested to find one capable of holding text.
- Replaced the `NULL` placeholder in the second position with a specific target string required by the challenge lab (`'X0Hp8e'`).
- The payload injected was URL-encoded as: `'+UNION+SELECT+NULL%2C+%27X0Hp8e%27%2C+NULL+--`.
- The application smoothly parsed the second column as a string type and successfully rendered the literal text value directly into the application interface, confirming column 2 accepts character data.

---

## Payloads Used
`'+UNION+SELECT+NULL%2C+%27X0Hp8e%27%2C+NULL+--`

---

## Result

The application successfully executed the payload and returned a standard `200 OK` HTTP response. The target string `'X0Hp8e'` was printed cleanly within the response interface, proving that the second column handles string content and is ready for data exfiltration.

---

## Impact

- Identification of an active injection channel for text data exfiltration.
- Drastically reduces attacker uncertainty, laying the groundwork for targeted extraction of schema configurations, usernames, or password hashes through the verified second column.

---

## Key Insight

A database strictly enforces data types on rows combined via a `UNION` statement. If an injection attempts to combine an integer column with a text literal, the database throws a fatal casting error, rendering a `500 Internal Server Error`. 

By substituting `NULL` flags one by one with a target string string token, an operator can pinpoint the exact positions in the application's layout engine that accept string properties, allowing subsequent database data exfiltration payloads to land reliably.

---

## Mitigation

- **Prepared Statements:** Implement strongly typed, parameterized queries to cleanly separate user data from command logic.
- **Input Filtering:** Apply strict server-side validation against incoming parameters using an allow-list model.
- **Defensive Error Handling:** Ensure all application exceptions are handled gracefully, preventing technical metadata or verbose server errors (`500 Internal Server Error`) from leaking structural clues to the browser.
- **Least Privilege:** Enforce the principle of least privilege on web-facing database connection strings to minimize the impact of data exposure.

---

## Evidence

- Screenshot 01 : Reflective extraction showing the requested string `'X0Hp8e'` printed cleanly within the user interface framework, marking the lab as solved.

**Lab Link:** https://portswigger.net/web-security/sql-injection/union-attacks/lab-find-column-containing-text
