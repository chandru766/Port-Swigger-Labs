# PortSwigger SQL Injection Labs - Comprehensive Notes

This document contains detailed notes, extracted workflows, and diagrams based on the PortSwigger SQL Injection lab diagrams.

---

## Lab 01: Product Category Filter

![Lab 01 Diagram](assets/lab1.png)

**Objective**: Display all products, including both released and unreleased products.
**Vulnerability**: SQL Injection (Boolean-based). The application uses a user-controlled `category` parameter directly inside an SQL query without properly sanitizing or parameterizing the input.

### Attack Flow

```mermaid
graph TD
    A[Category Input] --> B[SQL Query Formulation]
    B --> C[Input Changes Query Logic]
    C --> D[Filter Bypass]
    D --> E[(Database returns more records)]
```

**Key Technique**: 
Use a string terminator and an SQL comment (e.g., `' OR 1=1--`) to manipulate the boolean condition to always evaluate to `TRUE`. This bypasses the original `released = 1` logic.

**Remediation**: Use Prepared Statements / Parameterized Queries. Never concatenate untrusted user input directly into SQL queries.

---

## Lab 02: Login Functionality

![Lab 02 Diagram](assets/lab2.png)

**Objective**: Bypass authentication to access the administrator account.
**Vulnerability**: Authentication Bypass via SQLi. Username and password fields are concatenated directly into the authentication query.

### Attack Flow

```mermaid
graph TD
    A[Login Form] --> B[Username Parameter]
    B --> C[SQL Query]
    C --> D[Input Affects Query Logic]
    D --> E[Password Check Bypassed]
    E --> F[Administrator Access Granted]
```

**Key Technique**:
Provide an input like `administrator'--`. The `'` terminates the username string, and `--` comments out the password validation check, allowing login without knowing the password.

---

## Lab 03 & 04: Determining Number of Columns & Data Types

![Lab 03 & 04 Diagram](assets/lab3.png)

**Objective**: Determine the number of columns returned by the vulnerable query and find which columns are compatible with string data.
**Vulnerability**: UNION-based SQL Injection.

### Attack Methodology

```mermaid
graph LR
    A[SQLi Found] --> B[Understand UNION]
    B --> C[Test Column Count]
    C --> D[ORDER BY / NULL Test]
    D --> E[Confirm Count]
    E --> F[Test Data Types]
    F --> G[Identify String Column]
    G --> H[(Ready for Data Retrieval)]
```

**Key Concepts**:
1. **UNION Rules**: The injected query must return the exact same number of columns as the original query.
2. **Column Counting**: Use `ORDER BY 1`, `ORDER BY 2`, etc., until an error occurs, or use `UNION SELECT NULL, NULL...` increasing the count until a `200 OK` is returned.
3. **Data Type Matching**: Replace `NULL` with a string value (e.g., `'a'`) one by one to find which columns accept string data.

---

## Lab 05: Retrieving Data from Other Tables

![Lab 05 Diagram](assets/lab5.png)

**Objective**: Understand how UNION-based SQL injection can expose data from other tables.
**Vulnerability**: UNION-based SQL Injection.

### Attack Flow

```mermaid
graph TD
    A[SQLi Found] --> B[Determine Column Count]
    B --> C[2 Columns Confirmed]
    C --> D[Test Data Types]
    D --> E[Both Columns = STRING]
    E --> F[UNION SELECT Concept]
    F --> G[Query Target Table]
    G --> H[Data Displayed]
    H --> I[Administrator Account Identified]
```

**Key Technique**:
Once column count and data types are determined, formulate a payload like:
`' UNION SELECT username, password FROM users--`

---

## Lab 06: Retrieving Multiple Values Within a Single Column

![Lab 06 Diagram](assets/lab6.png)

**Objective**: Retrieve usernames and passwords when only ONE column is available for displaying useful text.
**Key Technique**: String Concatenation.

### Attack Flow

```mermaid
graph TD
    A[Retrieve Username] --> B[Retrieve Password]
    B --> C[Concatenate Values]
    C --> D[Separate with Delimiter]
    D --> E[Read Both Values Through One Column]
```

**Payload Example**:
`' UNION SELECT NULL, username || '~' || password FROM users--`
This joins the `username` and `password` with a `~` character, allowing both to be read from a single display column.

---

## Lab 07 & 08: Database Reconnaissance & Enumeration

![Lab 07 Diagram](assets/lab7.png)
![Lab 08 Diagram](assets/lab8.png)

**Objective**: Examine the database version and enumerate database structure (Tables and Columns).
**Vulnerability**: SQL Injection (Information Schema).

### Database Fingerprinting Flow

```mermaid
graph LR
    A[SQL Injection] --> B[Column Count]
    B --> C[Identify String Column]
    C --> D[Identify DBMS]
    D --> E[Select Version Query]
    E --> F[Database Version Output]
```

**Key Concepts**:
- **Information Schema**: Non-Oracle databases expose metadata through `information_schema.tables` and `information_schema.columns`.
- **Version Queries**: 
  - MySQL/Microsoft: `SELECT @@version`
  - PostgreSQL: `SELECT version()`
  - Oracle: `SELECT * FROM v$version`

---

## Lab 09: Blind SQL Injection (Conditional Responses)

![Lab 09 Diagram](assets/lab9.png)

**Objective**: Determine the administrator's password when the application does not directly display database results.
**Vulnerability**: Blind SQL Injection.

### Attack Logic

```mermaid
graph TD
    A[Unknown Password] --> B{Is Condition True?}
    B -- YES --> C[Continue / Record Match]
    B -- NO --> D[Try another value]
    C --> E[Determine Password Length]
    D --> E
    E --> F[Test Character Positions]
    F --> G[Compare Responses]
    G --> H[Reconstruct Password]
```

**Key Technique**:
Since data is not displayed, we rely on application behavior (e.g., whether a "Welcome back" message is shown) to answer TRUE/FALSE questions about the database. We first find the length, then iterate through characters using `SUBSTRING()`.

---

## Lab 10: Conditional Error-Based Blind SQLi

![Lab 10 Diagram](assets/lab10.png)

**Objective**: Infer database information through observable error behavior.
**Vulnerability**: Conditional Error-Based Blind SQLi.

### Error Inference Flow

```mermaid
graph TD
    A[SQL Injection] --> B[Conditional Test]
    B --> C{Is condition TRUE?}
    C -- YES --> D[Intentional DB ERROR]
    C -- NO --> E[NO ERROR]
    D --> F[Attacker Infers TRUE]
    E --> G[Attacker Infers FALSE]
```

**Key Technique**:
Instead of relying on different text on the page, the attacker uses SQL syntax that triggers a database error (like division by zero) *only* if the condition evaluates to TRUE. The presence or absence of a Server Error becomes the information channel.

---

### Universal Remediation
* **Use Prepared Statements (Parameterized Queries)**
* **Validate input where appropriate**
* **Apply least-privilege database accounts**
* **Do not expose detailed database error messages to users**
