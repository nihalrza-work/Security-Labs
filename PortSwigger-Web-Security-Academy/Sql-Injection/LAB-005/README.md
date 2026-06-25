# LAB-005: SQL Injection - Listing Database Contents (Non-Oracle)

## Objective

To identify, fingerprint, and exploit a SQL Injection vulnerability to navigate the database catalog, discover application-specific user tables, and extract the administrative credentials to achieve full authentication bypass.

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
- Mapped column data compatibility using type-agnostic NULL values (' UNION SELECT NULL, NULL -- '), confirming that the columns accepted text-based string payloads.

### 3. Database Isolation & Elimination (Fingerprinting)
A methodical, step-by-step dialect test was performed to isolate the backend engine:

- **Oracle Elimination:** Testing structural metadata behaviors against standard schema formats confirmed the backend lacked Oracle's structural requirements (such as the mandatory FROM DUAL clause or ALL_TABLES architecture), proving a non-Oracle ecosystem.
- **MySQL Disproval:** A spatial concatenation payload (Pets' AND 'ab' = 'a' 'b' -- ) failed to return products, proving that implicit spatial text concatenation was unsupported.
- **PostgreSQL Confirmation:** Testing the pipeline concatenation operator (Pets' AND 'ab' = 'a' || 'b' -- ) resulted in a normal page load with all products displayed. Because the database parsed the || syntax successfully to evaluate a true logic state, the backend was definitively confirmed as PostgreSQL.

### 4. Exploitation (Metadata Mapping & Data Extraction)
With PostgreSQL confirmed, a systematic data extraction process was executed:

- **Public Schema Discovery:** Queried the ANSI-standard system metadata table (information_schema.tables) while scoping exclusively to the application layer (table_schema = 'public') to isolate custom user tables from core system tables.
- **Table Identification:** Identified a non-default, application-generated table containing user metrics, structured with a randomized suffix: users_poyndo.
- **Column Names Enumeration:** Audited the information_schema.columns system space to extract the precise structural column names mapped inside the target table. This revealed the custom credentials storage attributes: username_lrnrow and password_sqqxun.
- **Admin Password Reveal:** Pivoted away from system metadata tables and directly queried the discovered columns from the active database table. This printed the full list of credentials straight to the webpage layout, exposing the administrator access token.

---

## Payloads Used

### Step 1: Mapping Tables in Public Schema
' UNION SELECT table_name, NULL FROM information_schema.tables WHERE table_schema = 'public' -- 

### Step 2: Extracting Target Columns
' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_schema = 'public' AND table_name = 'users_poyndo' -- 

### Step 3: Extracting Administrative Credentials
' UNION SELECT username_lrnrow, password_sqqxun FROM users_poyndo -- 

---

## Result

The application successfully executed the UNION queries, allowing complete visibility into the relational schema. The final extraction query extracted the custom username and password hashes, printing the administrator credentials onto the product page and allowing a successful administrative login to complete the lab.

---

## Impact

- Compromise of Data Confidentiality: Total unauthorized visibility into administrative and user records.
- Complete Authentication Bypass: Acquisition of plaintext or active credentials leading to direct privilege escalation.
- System-Wide Exposure: Full mapping of structural schema layouts, application architecture, and platform configuration details.

---

## Key Insight

In database systems like PostgreSQL, a UNION attack must respect both positional column counts and strict data-type constraints. PostgreSQL does not automatically juggle types (such as converting integers into text spaces implicitly); therefore, utilizing NULL as a starting universal solvent allows an operator to safely test boundaries. 

Once the data type lanes are established, mapping the standard information_schema structure allows an automated or manual tester to cleanly isolate application tables from internal system noise, providing an exact roadmap directly to sensitive transactional tables.

---

## Mitigation

- **Parameterized Queries:** Utilize prepared statements with strongly typed parameter bindings to ensure input parameters can never modify the runtime logic of the SQL statement.
- **Strict Input Validation:** Enforce an allow-list model on expected values (such as fixed product categories) at the server layer.
- **Disable Error Leaks:** Ensure database exception messages are neutralized at the application layer to block structural fingerprinting vectors.
- **Restrict Permissions:** Enforce the principle of least privilege on the application's connection string, ensuring it cannot access or read standard system metadata catalogs unnecessarily.

---

## Evidence
- Screenshot 01 : All tables in information schema displayed.
- Screenshot 02 : Target table in public schema.
- Screenshot 03 : Target columns reveal.
- Screenshot 04 : Admin password reveal.
- Screenshot 05 : Login as admin.

**Lab Link:** https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-non-oracle
