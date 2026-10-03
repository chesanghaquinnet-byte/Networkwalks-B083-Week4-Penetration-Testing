# Networkwalks-B083-Week4-Penetration-Testing
Authorized black-box penetration testing project for Mediroza General Hospital — Networkwalks Week 


# Networkwalks Week 4 Penetration Testing Project

## Client

**Mediroza General Hospital**

## Assessment Type

Authorized Black\-Box Web Application Penetration Test

## Scope

**Target:** `https://medirozahospital.com`

Testing was performed only against the authorized target as part of the Networkwalks internship project\.

1. Executive Summary

This project involved an authorized black-box penetration test of the Mediroza General Hospital web application.

The assessment followed a structured process:

1. Reconnaissance
2. Scanning and enumeration
3. Initial access
4. Password cracking
5. Deep reconnaissance
6. Findings and recommendations

Several security weaknesses were identified, including username enumeration, SQL injection, authentication bypass, unauthorized access to patient reports, weak PDF passwords, sensitive PDF metadata, an exposed backup directory, and sensitive information contained in an accessible database backup.

# 2\. Tools Used

The following tools and techniques were used during the assessment:

- WHOIS
- DNSRecon
- WhatWeb
- theHarvester
- CRT\.sh
- cURL
- Nmap
- Browser
- Networkwalks Hash Calculator
- John the Ripper
- qpdf
- ExifTool
- wget

# 3\. Methodology

The assessment followed this methodology:

**Footprinting → Scanning & Enumeration → Gaining Access → Password Cracking → Deep Reconnaissance → Findings & Recommendations**

---

# 4\. Reconnaissance

## 4\.1 WHOIS

WHOIS was used to gather domain registration information about the target\.

The information collected included the domain registrar, nameservers, creation date, and DNSSEC status\.

---

## 4\.2 DNS Reconnaissance

DNSRecon was used to gather DNS\-related information about the target domain\.

---

## 4\.3 Web Technology Enumeration

WhatWeb was used to identify technologies and server information associated with the target\.

The scan identified information such as the web server technology and other HTTP\-related details\.

4.4 HTTP Header and Website Enumeration

cURL was used to inspect HTTP responses and website resources.

The following commands were used:

curl -I https://medirozahospital.com

curl https://medirozahospital.com/robots.txt

The robots.txt file revealed the following paths:

/patient/
/staff/
/old/

###Evidence
![ robots.txt](robots.png)


# 5\. Milestone 1 — Initial Access

## 5\.1 Patient Login Page

The `/patient/` directory led to the patient login page:

```text
https://medirozahospital.com/patient/login.php
```

The authentication mechanism was tested to determine how the application responded to different usernames and passwords\.

---

## 5\.2 Username Enumeration

Different usernames were tested against the login form\.

The application returned different messages depending on whether the username existed\.

Examples included:

```text
Username not found
```

and:

```text
Incorrect password
```

This difference allowed valid usernames to be identified\.

### Finding

**Username Enumeration**

**Severity:** Medium

### Impact

An attacker could identify valid usernames and use them for further attacks\.

### Evidence 2 — Username Enumeration

![Username Enumeration](username.png)


6. SQL Injection and Authentication Bypass

6.1 SQL Injection Testing

The username field was tested with:

admin'

The application returned a SQL syntax error, indicating that user input was being incorporated into a database query.

────────

6.2 Authentication Bypass

The following authorized lab payload was then tested:

admin' --

The application accepted the input and provided access to the patient portal.

The portal displayed three patient reports.

Finding

SQL Injection / Authentication Bypass

Severity: Critical

Impact

An attacker could bypass the application’s authentication mechanism and gain access to restricted functionality.

Evidence 3 — SQL Injection Error
![Sql Injection Error](Sql-error.png)

Caption: SQL injection testing produced a database error, demonstrating insufficient input handling.

Evidence 4 — Authentication Bypass
![Authentication Bypass](sql-bypass.png)

Caption: The authentication bypass successfully opened the patient portal.


# 7\. Unauthorized Access to Patient Reports

After successfully bypassing authentication, the patient portal displayed:

```text
patient_report_1.pdf
patient_report_2.pdf
patient_report_3.pdf
```

The reports were downloaded for further analysis within the authorized assessment\.

### Finding

**Unauthorized Access to Patient Reports**

**Severity:** High

### Impact

Successful exploitation of the authentication vulnerability allowed access to confidential patient documents\.

### Evidence 5 — Patient Reports

![Evidence 5 - Patient Reports](patient-reports.png)

**Caption:** The patient portal displayed three confidential patient reports after the authentication bypass\.


# 8\. Milestone 2 — PDF Password Cracking

The downloaded PDF files were protected with passwords.

The PDF hashes were extracted and processed using the Networkwalks Hash Calculator.

The hashes began with:

$pdf$

John the Ripper was used with available wordlists to test the PDF passwords.

Two of the PDF passwords were recovered using the available wordlist.

The third PDF did not match the initial wordlist and required the John the Ripper wordlist.

The third PDF password was successfully recovered.

### Finding

Weak PDF Passwords

### Severity: High

### Impact

Weak document passwords allowed protected patient documents to be recovered using password-cracking techniques.

### Evidence 6 — PDF Hashes
![Pdf Hashes](pdf-hashes.png)

Caption: PDF password hashes were extracted for password-cracking analysis.

### Evidence 7 — PDF Password Cracking
![Paswword Cracking](password-cracking.png)

**Caption:** The PDF password was successfully recovered using a wordlist.


# 9\. PDF Decryption

The `qpdf` utility was used to decrypt the third PDF after recovering its password\.

The command used was:

```bash
qpdf --password="$PASSWORD" --decrypt patient_report_3.pdf report3_open.pdf
```

The decrypted file was successfully created as:

```text
report3_open.pdf
```


# 10\. Milestone 3 — PDF Metadata Analysis

ExifTool was used to inspect the metadata of the decrypted PDF.

The command used was:

exiftool report3_open.pdf

Important metadata discovered included:

Author   : j.malik
Comments : DB backup moved to /old before site migration, do not delete

This information provided a clue for further investigation of the /old/ directory.

### Finding

Sensitive Information in PDF Metadata

### Severity: Medium

### Impact

Metadata can unintentionally disclose internal usernames and operational information that may assist further attacks.

### Evidence 8 — PDF Decryption and Metadata
![Pdf Decryption and Metadata](pdf-decryption-metadata.png)

Caption: The decrypted PDF was inspected with ExifTool, revealing the author j.malik and an internal comment referencing the /old backup directory.


# 11\. Exposed Backup Directory

The `/old/` directory discovered during reconnaissance was accessed:

```text
https://medirozahospital.com/old/
```

Directory listing was enabled\.

The following database backup was discovered:

```text
mediroza_db_backup_2019.sql
```

### Finding

**Forgotten Backup Directory with Directory Listing**

**Severity:** Critical

### Impact

An attacker could discover and download sensitive files that should not be publicly accessible\.

### Evidence 10 — Exposed `/old/` Directory

![Evidence 10 - Exposed Backup Directory](old-directory.png)

**Caption:** Directory listing exposed the database backup file `mediroza_db_backup_2019.sql`\.


# 12\. Database Backup Download

The exposed database backup was downloaded using:

```bash
wget https://medirozahospital.com/old/mediroza_db_backup_2019.sql
```

The downloaded file was confirmed using:

```bash
ls -lh mediroza_db_backup_2019.sql
```

The file was successfully downloaded and was approximately 6\.2 KB\.

### Evidence 11 — Database Backup Download

![Evidence 11 - Database Backup Download](database-download.png)

**Caption:** The exposed SQL database backup was successfully downloaded for authorized analysis\.


# 13\. Database Backup Analysis

The SQL backup contained several database tables, including:

```text
staff
shareholders
```

---

## 13\.1 Staff Information

The `staff` table contained information including:

- Staff names
- Job titles
- Departments
- Email addresses
- Phone numbers
- National identification information
- Monthly salaries
- Employment dates

One record identified:

**Jameel Malik — IT Systems Administrator**

This corresponded with the PDF metadata:

```text
Author: j.malik
```

This provided a connection between the PDF metadata and the staff record in the exposed database backup\.

### Evidence 12 — Staff Database

![Evidence 12 - Staff Database](staff-database.png)

**Caption:** The exposed database backup contained staff records and sensitive employment information\.


## 13\.2 Shareholder Information

The `shareholders` table contained:

- Shareholder names
- Share percentages
- Number of shares
- Share classes

### Evidence 13 — Shareholders Database

![Evidence 13 - Shareholders Database](shareholders-database.png)

**Caption:** The exposed database backup contained shareholder information\.


### 14. Sensitive Database Information Exposure

The exposed database backup contained confidential organizational information, including staff salary information and shareholder information.

### Finding

Sensitive Database Information Exposure

### Severity: Critical

### Impact

Public exposure of the database backup could disclose confidential employee and organizational information.

Sensitive information such as national IDs, phone numbers, and personal email addresses should not be exposed publicly.

────────

15. Findings Summary

|#|Finding                                |Severity|
|-|---------------------------------------|--------|
|1|Username Enumeration                   |Medium  |
|2|SQL Injection / Authentication Bypass  |Critical|
|3|Unauthorized Access to Patient Reports |High    |
|4|Weak PDF Passwords                     |High    |
|5|Sensitive PDF Metadata                 |Medium  |
|6|Exposed Backup Directory               |Critical|
|7|Sensitive Database Information Exposure|Critical|

────────

# 16\. Recommendations

### 16.1 Prevent SQL Injection

Use prepared statements and parameterized queries instead of directly inserting user input into SQL queries.

Input validation should also be implemented.

────────

### 16.2 Prevent Username Enumeration

The application should return a generic authentication error such as:

Invalid username or password.

The application should not reveal whether a username exists.

────────

### 16.3 Strengthen Access Controls

Patient reports should only be accessible to properly authenticated and authorized users.

Authorization checks should be performed whenever a document is requested.

────────

### 16.4 Use Strong Document Passwords

Sensitive PDF documents should use strong, unique passwords.

Passwords should not be easily guessable or vulnerable to common wordlists.

────────

### 16.5 Remove Sensitive Metadata

Sensitive metadata should be removed from documents before publication or distribution.

Internal usernames and operational notes should not be unnecessarily embedded in documents.

────────

### 16.6 Remove Public Backup Files

Database backups should never be stored inside publicly accessible web directories.

The /old/ directory should be removed or properly restricted.

Directory listing should also be disabled.

────────

### 16.7 Secure Database Backups

Database backups should be stored in a secure location with appropriate access controls.

Backups should not be publicly accessible and should not unnecessarily contain sensitive information.

────────

# 17\. Conclusion

The assessment demonstrated how multiple security weaknesses could be identified and chained together during an authorized black-box penetration test.

The assessment began with reconnaissance and progressed through authentication testing, SQL injection testing, document access, password recovery, PDF metadata analysis, and investigation of an exposed database backup.

The findings demonstrate the importance of secure input handling, strong authentication controls, proper authorization, secure document management, protection of backup files, and appropriate handling of sensitive information.

All testing was performed within the authorized scope of the Networkwalks internship project.



# 👤 Author

**Chesangha Quinneta**

**Networkwalks 2026 Intern**

LinkedIn: [https://www\.linkedin\.com/in/cyber\~\-nneta\-77a37b3ab?utm\_source=share\_via&utm\_content=profile&utm\_medium=member\_ios](https://www.linkedin.com/in/cyber~-nneta-77a37b3ab?utm_source=share_via&utm_content=profile&utm_medium=member_ios)

---

## Project Information

**Program Name:** Cybersecurity at Networkwalks \| **Week:** 04 \| **Project:** Cybersecurity & Black\-Box Penetration Testing \| **Repository:** GitHub


