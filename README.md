# Penetration Testing Report
**Mediroza General Hospital**

**Engagement Type**   : Black-box Penetration Test  
**Target**            : https://medirozahospital.com  
**Client**            : Mediroza General Hospital  
**Batch / Week**      : B084 | Week 4  
**Duration**          : 5 Days  
**Tester**            : Khairunnisya Lubis  
**Date**              : 24 November 2026  
**Authorization**     : Written authorization provided by Networkwalks Academy  

---

## 1. Executive Summary

A black-box penetration test was conducted against the Mediroza General Hospital web application as part of an authorized training exercise.

The assessment identified several high and critical severity vulnerabilities that could be chained together. Key findings include:

- SQL Injection authentication bypass leading to unauthorized access to the Patient Portal
- Retrieval of three confidential patient laboratory reports
- Successful recovery of weak PDF encryption passwords
- Public exposure of an internal database backup containing sensitive human resources and ownership data

The most critical finding was an unauthenticated database backup file that exposed employee compensation details and shareholder information.

**Overall Risk Rating: Critical**

Immediate remediation of the identified issues is recommended.

---

## 2. Scope and Methodology

### Scope
- Target: `https://medirozahospital.com`
- Type: Full black-box web application penetration test
- Authorization: Written authorization from Networkwalks

### Rules of Engagement
- Testing limited to the target domain only
- No social engineering
- No denial-of-service testing
- No testing outside the approved scope

### Methodology
1. Reconnaissance (robots.txt, directory enumeration)
2. Authentication and session testing
3. Input validation testing
4. File and metadata analysis
5. Sensitive data exposure assessment

---

## 3. Findings

### Finding 1 – Directory Listing & Sensitive Path Disclosure
**Severity: High**

The application disclosed sensitive paths via `robots.txt` and had directory listing enabled on multiple directories (`/patient/`, `/staff/`, and `/old/`).

This allowed easy discovery of application structure and sensitive files without authentication.

---

### Finding 2 – SQL Injection Authentication Bypass
**Severity: Critical**

The Patient Portal login functionality was vulnerable to SQL Injection. A simple authentication bypass payload granted full access to the restricted portal without valid credentials.

**Impact:** Unauthorized access to confidential patient laboratory reports.

---

### Finding 3 – Unauthorized Access to Patient Laboratory Reports
**Severity: High**

Three confidential pathology reports were successfully retrieved from the Patient Portal after authentication bypass.

The reports contained protected health information (PHI) and were password-protected.

---

### Finding 4 – Weak PDF Encryption
**Severity: High**

All three laboratory report PDFs used weak, easily crackable passwords. The passwords were recovered using standard dictionary attacks.

---

### Finding 5 – Critical Sensitive Data Exposure
**Severity: Critical**

A publicly accessible database backup was discovered at:


The file required no authentication and contained:

- **Staff records** (30 employees) including full names, job titles, departments, contact details, national IDs, and **monthly salaries**.
- **Shareholder records** (10 entries) including ownership percentages and share classes.

**Summary of Exposed Data:**
- Highest monthly salary observed: R160,000
- Lowest monthly salary observed: R19,000
- Total staff records exposed: 30
- Total shareholder records exposed: 10

This represents a critical breach of both employee privacy and corporate confidentiality.

---

## 4. Risk Rating Summary

| Finding | Severity | CWE ID | CWE Name |
| --- | --- | --- | --- |
| SQL Injection Authentication Bypass | Critical | CWE-89 | Improper Neutralization of Special Elements used in an SQL Command ('SQL Injection') |
| Public Database Backup (Staff + Shareholders) | Critical | CWE-200 / CWE-538 | Exposure of Sensitive Information to an Unauthorized Actor / Insertion of Sensitive Information into Externally-Accessible File or Directory |
| Unauthorized Access to Patient Reports | High | CWE-284 / CWE-862 | Improper Access Control / Missing Authorization |
| Weak PDF Password Protection | High | CWE-521 / CWE-326 | Weak Password Requirements / Inadequate Encryption Strength |
| Directory Listing Enabled | High | CWE-548 | Exposure of Information Through Directory Listing |
| Sensitive Path Disclosure (robots.txt) | Medium | CWE-200 / CWE-538 | Exposure of Sensitive Information to an Unauthorized Actor |

**Overall Risk: Critical**

---

## 5. Recommendations

### Immediate (Critical Priority)
1. Remove public access to the `/old/` directory immediately.
2. Delete or securely relocate the database backup file.
3. Disable directory listing across the entire web server.

### Short-term
4. Remediate the SQL Injection vulnerability using parameterized queries / prepared statements.
5. Implement generic authentication error messages.
6. Strengthen PDF protection or replace password-based protection with proper access control.
7. Strip sensitive metadata from documents before distribution.

### Long-term
8. Implement a formal data classification and backup policy.
9. Conduct regular security assessments.
10. Consider deploying a Web Application Firewall (WAF).

---

## 6. Conclusion

The assessment demonstrated that multiple weaknesses in the Mediroza General Hospital web application could be chained to achieve significant unauthorized access to both medical and corporate sensitive data.

The public exposure of employee salary information and shareholder details is particularly concerning and warrants immediate attention.

All findings should be addressed according to the priority indicated above.

---

**Disclaimer**  
This report was prepared as part of an authorized educational penetration testing exercise. All activities were conducted within the approved scope and under written authorization from Networkwalks Academy. The data referenced in this report is fictional and exists solely for training purposes.

---

**End of Report**

