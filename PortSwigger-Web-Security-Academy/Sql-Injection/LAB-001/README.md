# LAB-001: SQL Injection - View Unreleased Products via WHERE Clause Manipulation

## Objective

To identify and exploit a SQL Injection vulnerability that allows viewing unreleased products by manipulating the WHERE clause logic of the backend SQL query.

---

## Vulnerability Type

SQL Injection

---

## Target

Product listing page (filter parameter controlling visibility of released/unreleased products)

---

## Context

The application uses a SQL query with a WHERE clause to filter and display only released products.  
User input is directly injected into this query without proper sanitization or parameterization, allowing modification of query logic.

---

## Approach

### 1. Observation
- Identified input parameter controlling product visibility
- Noticed only “released” products were shown by default
- Inferred backend filtering using a WHERE clause condition

### 2. Injection Testing
- Tested input field for SQL injection behavior
- Observed that modifying logical conditions affected output

### 3. Exploitation
- Injected payload to alter WHERE clause logic
- Forced condition to always evaluate as true
- Bypassed filter restricting unreleased products

---

## Payloads Used
' OR 1=1--


(URL encoded form used in request: `'+OR+1=1+--`)

---

## Result

Application displayed both released and unreleased products by bypassing the intended filtering mechanism.

---

## Impact

- Unauthorized access to hidden/unreleased product data  
- Bypass of business logic controlling product visibility  
- Exposure of sensitive inventory information

---

## Key Insight

SQL Injection can directly manipulate backend query logic when user input is concatenated into SQL statements without proper validation.

Even simple boolean manipulation (`OR 1=1`) can fully bypass application-level filters.

---

## Mitigation

- Use parameterized queries (prepared statements)
- Avoid dynamic SQL query construction
- Enforce strict input validation where applicable
- Apply least-privilege database access controls

---

## Evidence
- Screenshot after payload injection showing all products
