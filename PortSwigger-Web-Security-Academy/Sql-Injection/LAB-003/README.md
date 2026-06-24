# LAB-003: SQL Injection - Querying Database Type and Version (Oracle)

## Objective

To identify and exploit a SQL Injection vulnerability that allows disclosure of the backend database type and version information by leveraging a UNION-based query.

---

## Vulnerability Type

SQL Injection (UNION-based)

---

## Target

Category filter parameter vulnerable to SQL Injection and capable of reflecting query results in the application's response.

---

## Context

The application concatenates user input directly into an SQL query without proper sanitization or parameterization. Since the query results are reflected in the response, a UNION-based SQL Injection can be used to retrieve information from Oracle system views.

---

## Approach

### 1. Observation

- Identified a parameter vulnerable to SQL Injection
- Confirmed that database query results were reflected in the response
- Determined that a UNION-based attack was feasible

### 2. Column Enumeration

- Determined that the original query returned two columns
- Used `NULL` as a placeholder to maintain column compatibility

### 3. Exploitation

- Queried Oracle's `v$version` dynamic performance view
- Retrieved data from the `BANNER` column containing version information
- Successfully identified the backend database as Oracle

---

## Payloads Used
' UNION SELECT NULL, BANNER FROM v$version --

## Result

The application returned Oracle version information from the `v$version` view, successfully disclosing the backend database type and version details.

---

## Impact

- Database fingerprinting and technology disclosure
- Exposure of version-specific information
- Enables identification of known Oracle vulnerabilities
- Assists further targeted exploitation and reconnaissance

---

## Key Insight

Oracle stores version information in the `v$version` system view. When user input is directly concatenated into SQL queries, attackers can leverage UNION-based SQL Injection to retrieve sensitive database metadata.

Even simple version disclosure significantly assists attackers by reducing uncertainty and enabling targeted attacks.

---

## Mitigation

- Use parameterized queries (prepared statements)
- Avoid dynamic SQL query construction
- Implement strict server-side input validation
- Suppress unnecessary database information disclosure
- Apply least-privilege permissions to database accounts

---

## Evidence

- Screenshot showing Oracle version information returned after payload execution

**Lab Link:** https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-oracle
