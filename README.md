# Penetration Testing Report
**Mediroza General Hospital**

| Item                    | Detail                                      |
|-------------------------|---------------------------------------------|
| **Engagement Type**     | Black-box Penetration Test                  |
| **Target**              | https://medirozahospital.com                |
| **Client**              | Mediroza General Hospital                   |
| **Batch / Week**        | B083 | Week 4                               |
| **Duration**            | 5 Days                                      |
| **Tester**              | Khairunnisya Lubis                          |
| **Date**                | 24 November 2026                            |
| **Authorization**       | Written authorization provided by Networkwalks Academy |

---

## 1. Executive Summary

A black-box penetration test was conducted against the Mediroza General Hospital web application as part of an authorized training exercise by Networkwalks Academy.

The assessment successfully identified and exploited a chain of vulnerabilities that led to complete compromise of confidential patient data and internal corporate information.

**Key Findings:**
- Critical SQL Injection authentication bypass on the Patient Portal
- Unauthorized retrieval of 3 confidential patient laboratory reports (PHI)
- Successful recovery of weak PDF encryption passwords
- Public exposure of an internal database backup containing **staff salaries** and **shareholder details**

The most critical impact was the unauthenticated exposure of a complete database backup containing the monthly salaries of 30 employees and the full ownership structure of the hospital.

**Overall Risk Rating: Critical**

Immediate remediation is strongly recommended.

---

## 2. Scope and Methodology

### 2.1 Scope
- **Target**      : https://medirozahospital.com
- **Type**        : Full black-box web application penetration test
- **Authorization**: Written authorization from Networkwalks Academy

### 2.2 Rules of Engagement
- Testing limited to the target domain only
- No social engineering
- No denial-of-service testing
- No testing outside the approved scope

### 2.3 Methodology
1. Reconnaissance (`robots.txt`, directory enumeration)
2. Authentication mechanism analysis
3. Input validation testing (SQL Injection)
4. Document retrieval and cryptographic analysis
5. Metadata analysis
6. Sensitive data exposure assessment

---

## 3. Findings and Proof of Exploitation

### Finding 1 – Sensitive Path Disclosure via robots.txt
**Severity: Medium**  
**CWE: CWE-200**

**Description**  
The file `https://medirozahospital.com/robots.txt` disclosed three sensitive directories:
User-agent: *

Disallow: /patient/
Disallow: /staff/
Disallow: /old/


**Impact**  
Attackers can easily map the application structure without scanning.

**Evidence**  
- Screenshot of robots.txt content

---

### Finding 2 – SQL Injection Authentication Bypass
**Severity: Critical**  
**CWE: CWE-89**

**Description**  
The login form at `/patient/login.php` was vulnerable to SQL Injection.  

**Payload used:**

Username: admin' --
Password: (any value)


This successfully bypassed authentication and granted access to the Patient Portal.

**Impact**  
Full unauthorized access to the restricted Patient Portal containing confidential laboratory reports.

**Evidence**
- Screenshot of successful login after payload
- Screenshot of Patient Portal (`/patient/portal.php`) showing 3 laboratory reports

---

### Finding 3 – Unauthorized Access to Patient Laboratory Reports
**Severity: High**

**Description**  
After authentication bypass, three confidential pathology reports were retrieved:

1. Pathology Report – S. Dlamini
2. Pathology Report – P. Reddy
3. Pathology Report – E. Thompson

**Evidence**
- Screenshots of the three downloaded PDF files
- Screenshot of the portal page listing the reports

---

### Finding 4 – Weak PDF Encryption
**Severity: High**  
**CWE: CWE-326 / CWE-521**

**Description**  
All three laboratory report PDFs used weak passwords that were successfully recovered using dictionary attacks.

| File                        | Recovered Password |
|----------------------------|--------------------|
| patient_report_1.pdf       | `123456`           |
| patient_report_2.pdf       | `password`         |
| patient_report_3.pdf       | `!@#$%^&`          |

**Evidence**
- Hash extraction using `pdf2john`
- Successful decryption using `qpdf`

---

### Finding 5 – Critical Sensitive Data Exposure (Database Backup)
**Severity: Critical**  
**CWE: CWE-200 / CWE-538**

**Description**  
Metadata analysis of the decrypted PDF (`opened_3.pdf`) revealed the following comment:

Comments: DB backup moved to /old before site migration, do not delete
Author  : j.malik

This led to the discovery of a publicly accessible database backup:
[medirozahospital.com/old/mediroza_db_backup_2019.sql](https://medirozahospital.com/old/mediroza_db_backup_2019.sql)


The file required **no authentication** and contained:

#### Staff Records (30 employees)
- Full name, job title, department, email, phone, national ID, **monthly salary (ZAR)**, date joined

**Notable entries:**
- Highest salary : Dr. Johan van der Merwe (Medical Director) – **R160,000**
- IT Systems Administrator (j.malik) – **R58,000**
- Lowest salary  : Linda Fourie (Receptionist) – **R19,000**

#### Shareholder Records (10 entities)
| Shareholder                        | Share % | Share Class    |
|------------------------------------|---------|----------------|
| Dr. Rajesh Naidoo                  | 18.0%   | Ordinary       |
| Cedar Health Holdings (Pty) Ltd    | 15.0%   | Ordinary       |
| Dr. Johan van der Merwe            | 12.0%   | Ordinary       |
| Reddy Family Trust                 | 11.0%   | Ordinary       |
| Thabo Molefe                       | 10.0%   | Ordinary       |
| Sarah Botha                        | 9.0%    | Ordinary       |
| Dr. Ahmed Kara                     | 8.0%    | Preferential   |
| Naledi Zulu                        | 7.0%    | Ordinary       |
| Michael Roberts                    | 6.0%    | Ordinary       |
| Dr. Vikram Chetty                  | 4.0%    | Preferential   |

**Impact**  
Complete exposure of employee compensation data and corporate ownership structure.

**Evidence**
- Screenshot of metadata (`exiftool`)
- Screenshot of directory listing `/old/`
- Partial content of the SQL dump showing staff and shareholder tables

---

## 4. Risk Rating Summary

| # | Finding                                      | Severity  | CWE          |
|---|----------------------------------------------|-----------|--------------|
| 1 | SQL Injection Authentication Bypass          | Critical  | CWE-89       |
| 2 | Public Database Backup (Staff + Shareholders)| Critical  | CWE-200/538  |
| 3 | Unauthorized Access to Patient Reports       | High      | CWE-284/862  |
| 4 | Weak PDF Password Protection                 | High      | CWE-326/521  |
| 5 | Directory Listing Enabled                    | High      | CWE-548      |
| 6 | Sensitive Path Disclosure (robots.txt)       | Medium    | CWE-200      |

**Overall Risk Rating: Critical**

---

## 5. Recommendations and Remediation

### Immediate (Critical)
1. Immediately remove public access to the `/old/` directory
2. Delete or move the database backup file to a secure location
3. Disable directory listing on the web server

### Short-term
4. Fix the SQL Injection vulnerability using **parameterized queries / prepared statements**
5. Implement generic error messages to prevent username enumeration
6. Replace weak PDF password protection with proper access control (or stronger encryption)
7. Strip sensitive metadata from all documents before distribution

### Long-term
8. Implement formal data classification and secure backup policies
9. Conduct regular security assessments
10. Consider deploying a Web Application Firewall (WAF)

---

## 6. Conclusion

The penetration test demonstrated that multiple vulnerabilities in the Mediroza General Hospital web application could be successfully chained to achieve unauthorized access to both Protected Health Information (PHI) and highly sensitive corporate data (employee salaries and shareholder structure).

The most severe finding was the publicly accessible database backup. This issue alone warrants immediate attention.

All findings should be remediated according to the priority indicated in this report.

---

**Disclaimer**  
This report was prepared as part of an authorized educational penetration testing exercise conducted under written authorization from Networkwalks Academy. All data referenced is fictional and exists solely for training purposes.

**End of Report**

