# LAB-006: SQL Injection - Listing Database Contents (Oracle)

## Objective

To identify, fingerprint, and exploit a SQL Injection vulnerability to navigate the Oracle data dictionary catalog, discover application-specific user tables, and extract the administrative credentials to achieve full authentication bypass.

---

## Vulnerability Type

SQL Injection (UNION-based)

---

## Target

Category filter parameter vulnerable to SQL Injection and capable of reflecting query results in the application's response.

---

## Context

The application concatenates user input directly into a dynamic SQL execution string without sanitization or data typing. Since the results of the query are loops-processed and directly rendered onto the HTML layout, a UNION-based attack allows an attacker to map system metadata catalogs, target specific transactional tables, and extract administrative secrets.

---

## Approach

### 1. Observation
- Identified that the product category filter parameter was vulnerable to SQL Injection.
- Confirmed that database rows returned by the query were directly injected into the frontend interface.

### 2. Column Enumeration
- Determined that the original query expected a structural layout of exactly two columns.
- Mapped column data compatibility using type-agnostic NULL values. In Oracle, testing required appending a valid table source, confirming that the columns accepted text-based string payloads.

### 3. Database Isolation & Elimination (Fingerprinting)
A methodical, step-by-step dialect test was performed to isolate the backend engine:

- **Non-Oracle Elimination (PostgreSQL/MySQL/MSSQL):** Initial probing with a simple subquery payload (' AND (SELECT 1) = 1 -- ') triggered an immediate **Internal Server Error (HTTP 500)**. Since standard relational databases allow selecting literals without a source table, their involvement was ruled out.
- **Oracle Confirmation (The Dual/Metadata Test):** The application failed to process queries lacking a source table because Oracle strictly requires a FROM clause for all SELECT operations. Injecting the Oracle-specific dummy table (' AND (SELECT 1 FROM dual) = 1 -- ') or querying system data frames resolved successfully with an HTTP 200 OK. This behavior conclusively fingerprinted the database engine as **Oracle**.

### 4. Exploitation (Metadata Mapping & Data Extraction)
With Oracle confirmed, a systematic data extraction process was executed using its unique Data Dictionary Architecture:

- **Table Identification via Data Dictionary:** Queried Oracle's public metadata view (all_tables). Due to the strict requirements of Oracle's search engine, the table tracking criteria had to be filtered using strict **UPPERCASE** text constraints. This successfully revealed a non-default, application-generated table containing user metrics, structured with a randomized suffix: USERS_AAEIRL.
- **Column Names Enumeration:** Audited the all_tab_columns metadata view to extract the precise structural column names mapped inside the target table. Column swapping payloads (' UNION SELECT NULL, column_name...') were used to find the specific layout lane used by the application frontend to display text. This uncovered the custom credentials storage attributes: USERNAME_XXXXXX and PASSWORD_XXXXXX.
- **Admin Password Reveal:** Pivoted away from system metadata views and directly queried the discovered columns from the active database table. This printed the full list of credentials straight to the webpage layout, exposing the administrator access token.

---

## Payloads Used

### Step 1: Mapping Tables via Oracle Data Dictionary
' UNION SELECT table_name, NULL FROM all_tables -- 

### Step 2: Extracting Target Columns (Strict Uppercase Filtering)
' UNION SELECT NULL, column_name FROM all_tab_columns WHERE table_name = 'USERS_AAEIRL' -- 

### Step 3: Extracting Administrative Credentials
' UNION SELECT USERNAME_XXXXXX, PASSWORD_XXXXXX FROM USERS_AAEIRL -- 
(Note: Replace USERNAME_XXXXXX and PASSWORD_XXXXXX with the exact column strings revealed in Step 2).

---

## Result

The application successfully executed the UNION queries, allowing complete visibility into the Oracle schema. The final extraction query pulled the custom username and password hashes, printing the administrator credentials onto the product page and allowing a successful administrative login to complete the lab.

---

## Impact

- **Compromise of Data Confidentiality:** Total unauthorized visibility into administrative and user records.
- **Complete Authentication Bypass:** Acquisition of plaintext or active credentials leading to direct privilege escalation.
- **System-Wide Exposure:** Full mapping of structural schema layouts, database engine properties, and platform configuration details.

---

## Key Insight

In database systems like Oracle, a UNION attack must respect both positional column counts and strict data-type constraints, alongside database-specific syntax traits like the mandatory FROM clause. Oracle does not automatically juggle types, and strings passed into its system catalog search filters (such as table_name) are strictly **case-sensitive**. Failing to supply uppercase text strings causes queries to return zero rows silently. 

Once these syntax lanes are established, utilizing system views like all_tables and all_tab_columns while leveraging proper rendering columns provides a clean, reliable path to sensitive transactional data.

---

## Mitigation

- **Parameterized Queries:** Utilize prepared statements with strongly typed parameter bindings to ensure input parameters can never modify the runtime logic of the SQL statement.
- **Strict Input Validation:** Enforce an allow-list model on expected values (such as fixed product categories) at the server layer.
- **Disable Error Leaks:** Ensure database exception messages are neutralized at the application layer to block structural fingerprinting vectors.
- **Restrict Permissions:** Enforce the principle of least privilege on the application's connection string, ensuring it cannot access or read standard system metadata catalogs (all_tables, all_tab_columns) unnecessarily.

---

## Evidence

- Screenshot 01 : Oracle syntax satisfied.
- Screenshot 02 : Product page displayed for oracle metadata.
- Screenshot 03 : Two columns exist for UNION ATTACK.
- Screenshot 04 : Table names in oracle metadata revealed.
- Screenshot 05 : Target table identified.
- Screenshot 06 : Target columns.
- Screenshot 07 : Admin password reveal.
- Screenshot 08 : Admin login.

**Lab Link:** https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-oracle
