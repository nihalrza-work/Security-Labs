# LAB-011: Blind SQL Injection with Conditional Responses

## Objective

To identify and exploit a blind SQL Injection vulnerability on a PostgreSQL backend by analyzing conditional web application behavior ("Welcome back" message presence), leveraging automated string character enumeration via Burp Suite Intruder to reconstruct an administrative password.

---

## Vulnerability Type

SQL Injection (Blind / Conditional Responses)

---

## Target

Tracking cookie/parameter vulnerable to SQL Injection where execution feedback is leaked exclusively through binary conditional UI modifications rather than direct data reflection.

---

## Context

Unlike `UNION`-based injection flaws where results are explicitly rendered to screen layout elements, this application runs queries silently in the backend. If the resulting SQL context returns one or more matching rows, a standard string expression (`Welcome back`) is appended to the UI header. If the condition evaluates to empty or false, the element is completely dropped. This behavioral variance serves as a single-bit oracle to exfiltrate database cell contents character-by-character.

---

## Approach

### 1. Observation & Oracle Baseline Setup
* Monitored application responses during logical parameter alterations.
* Injecting a fundamentally `TRUE` statement (`' AND 1=1 --`) resulted in the appearance of a standard validation banner (`Welcome back`).
* Injecting a fundamentally `FALSE` statement (`' AND 1=2 --`) forced the validation indicator to vanish entirely from the UI frame. This confirmed the presence of a viable blind SQL injection oracle.

### 2. Structural Mapping & Column Enumeration
* **Query Column Layout:** Verified structural layouts using sorting benchmarks. The injection of `ORDER BY 1` executed without an error, whereas an intentional shift to `ORDER BY 2` produced an application fault, proving that the underlying backend application tracking block returns exactly one column.
* **Data Type Definition:** Appending a text literal into the isolated column rendered cleanly, confirming that the singular returning position was compatible with string handling variables.

### 3. Database Isolation & Elimination (Fingerprinting)
To track and identify the specific backend execution engine without triggering invalid syntax termination states, an architectural behavioral check was performed:

* **Oracle Elimination:** Testing system table lookups using generic layout targets did not require an external standalone dummy view mapping (`FROM dual`), indicating a non-Oracle server environment.
* **Metadata Table Verification:** Executed an explicit `EXISTS` pattern map to check for standard operational frameworks: `' AND EXISTS (SELECT * FROM information_schema.tables) --`. The persistence of the `Welcome back` validation banner proved the existence of the metadata views, ruling out isolated standalone systems.
* **PostgreSQL Confirmation vs. MySQL:** Tested native string concatenation behaviors. Passing PostgreSQL's characteristic pipe string joining sequence (`'abc' || ''`) kept the conditional message active on screen, while attempting engine-specific space rules or alternative concatenation formatting patterns dropped the truth evaluation. This definitively fingerprinted the engine as PostgreSQL.

### 4. Schema Mapping & Pattern Discovery
* **Table Discovery:** Used pattern matching operators (`LIKE`) to narrow down existing structures sequentially. Injecting tests for table labels beginning with the letter 'u' (`' AND EXISTS (SELECT * FROM information_schema.tables WHERE table_name LIKE 'u%') --`) returned a `TRUE` state, which eventually led to locating the standard target authentication source `users`.

### 5. Automated Exploitation via Burp Suite Intruder
* Because direct multi-row reading was restricted by the blind behavior, character-by-character validation was required.
* **Password Length Mapping:** Validated character limits using the native size tracking function: `' AND (SELECT LENGTH(password) FROM users WHERE username='administrator')=20 --`. The application evaluated this as `TRUE`, establishing a strict 20-character target string baseline.
* **Character Brute Forcing Strategy:** Set up an automation payload array in Burp Suite Intruder targeted at substring index processing positions. Since SQL character offsets utilize 1-based indexing structures, the checking function was configured to iterate through indices 1 through 20.
* **Payload Applied:**
  `TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§a§`
* **Execution Output:** Automated matching against character vectors captured valid hits based on response size and string reflections. The full administrative token was reconstructed character-by-character: `di61wwk6yv0msy2pd43c`.
* **Authentication:** Navigated to the portal endpoint, entered `administrator` alongside the brute-forced secret token, and confirmed a successful login state.

---

## Payloads Used

`' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a' --`

---

## Result

The application successfully processed the automated positional evaluation, leaking the validation string state uniquely when the target character matched the backend variable. The total string layout was completely mapped out to yield the 20-character admin credentials (`administrator` / `di61wwk6yv0msy2pd43c`), allowing full target account access.

---

## Impact

* Full administrative infrastructure compromise via secondary blind execution.
* Bypasses traditional visibility restrictions, proving data leaks can manifest through simple structural UI adjustments.
* Complete horizontal data exfiltration through automated truth testing logic.

---

## Key Insight

Blind SQL Injection vulnerabilities emphasize that an application does not need to print database text directly on a web layout to leak confidential information. As long as input modifications can alter the execution logic of the query—causing a visible change in the page architecture, like the appearance or disappearance of a "Welcome back" element—the interface acts as a single-bit leakage point.

By wrapping extraction statements in a tracking substring evaluation function like `SUBSTRING(column, start, length)` and measuring the structural truth values via automation tools like Burp Suite Intruder, an external operator can translate a series of simple true/false UI answers into a full textual password string.

---

## Mitigation

* **Prepared Statements:** Implement strongly typed, parameterized queries to cleanly separate user data from command logic.
* **Input Filtering:** Apply strict server-side validation against incoming parameters using an allow-list model.
* **Defensive Error Handling:** Ensure all application exceptions are handled gracefully, preventing technical metadata or verbose server errors (`500 Internal Server Error`) from leaking structural clues to the browser.
* **Least Privilege:** Enforce the principle of least privilege on web-facing database connection strings to minimize the impact of data exposure.

---

## Evidence

* Sequential true/false states proving logical structural control, with successful brute force outputs identifying the exact administrative access password (`di61wwk6yv0msy2pd43c`).
* Referenced files: 
  * `01-welcome-back-appear-for-true-query.png`
  * `02-welcome-back-disappear-for-false-query.png`
  * `03-subquery-works-can-access-information-schema.png`
  * `04-information-schema-exists-oracle-eliminated.jpg`
  * `05-MySQL-eliminated-with-welcome-msg-disappear-on-concatenation-syntax.jpg`
  * `06-PostgreSQL-confirmed-with-welcome-back-msg-appear-on-concatenation.jpg`
  * `07-only-one-column-exists-determined-by-order-by.png`
  * `08-the-only-column-that-exist-contains-string.png`
  * `09-table-with-name-like-u%-exists.png`
  * `10-table-name-like-users%-exist.jpg`
  * `11-welcome-back-disappear-on-length-of-table-name-like-users-%3E-5.png`
  * `12-users-table-exist-confirmed.jpg`
  * `13-three-columns-exist-in-users-table.jpg`
  * `14-column-name-username-exist-in-table-users.jpg`
  * `15-column-name-password-exists-in-table-users.jpg`
  * `16-administrator-username-exists-in-table-users.jpg`
  * `17-administartor-password-length-found.png`
  * `18-bruteforcing-admin-password-one-character-at-a-time.png`
  * `19-first-letter-of-the-admin-password-found.jpg`
  * `20-last-character-of-admin-password-found.jpg`
  * `21-login-as-administrator-successful.png`

**Lab Link:** https://portswigger.net/web-security/sql-injection/blind/lab-conditional-responses
