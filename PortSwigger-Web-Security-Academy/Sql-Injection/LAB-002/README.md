# LAB-002: SQL Injection - Authentication Bypass via Login Input Manipulation

## Objective

To exploit a SQL Injection vulnerability in the login functionality to bypass authentication and gain unauthorized access.

---

## Vulnerability Type

SQL Injection

---

## Target

Login form (username and password input fields)

---

## Context

The application constructs a SQL query using user-supplied login credentials without proper sanitization.  
This allows manipulation of query logic to bypass authentication checks.

---

## Approach

### 1. Observation
- Identified login form as primary input point
- Observed backend likely validating credentials via SQL query
- Noted absence of input filtering or sanitization

### 2. Injection Testing
- Tested SQL injection payload in username field
- Observed login bypass behavior when logical condition altered

### 3. Exploitation
- Injected payload to manipulate WHERE clause logic
- Commented out password validation part of query

---

## Payloads Used
' OR 1=1 --


(Injected in username field)

---

## Result

Authentication was bypassed successfully, allowing login without valid credentials.

---

## Impact

- Unauthorized access to application accounts
- Complete bypass of authentication mechanism
- Potential exposure of sensitive user data

---

## Key Insight

SQL Injection in authentication logic can allow an attacker to bypass login mechanisms by forcing SQL conditions to evaluate as true.

Even simple boolean manipulation can completely break authentication systems.

---

## Mitigation

- Use parameterized queries (prepared statements)
- Never concatenate user input in authentication queries
- Implement secure authentication frameworks
- Apply strict input validation
- Use proper password hashing mechanisms

---

## Evidence

- Screenshot of login page before injection
- Screenshot showing successful bypass after payload execution
