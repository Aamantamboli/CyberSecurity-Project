# OWASP Juice Shop Penetration Testing Report

<img src="https://raw.githubusercontent.com/juice-shop/juice-shop/master/frontend/src/assets/public/images/JuiceShop_Logo.png"
     alt="Juice Shop" width="120">

This repository contains the **penetration testing report** for the [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/), a deliberately insecure web application designed to practice and learn web application security.

## 🔍 Objective
The main goal of this project was to identify and exploit vulnerabilities in OWASP Juice Shop based on the **OWASP Top 10** categories, document findings, and provide remediation steps.

## 📑 Summary of Findings
Below are the key vulnerabilities identified during the penetration test:

1. **SQL Injection – Authentication Bypass**  
   - Login endpoint vulnerable to SQL injection payloads.  
   - **Impact:** Unauthorized access to user/admin accounts.  
   - **Severity:** Critical (CVSS 9.8)  

2. **Insecure Transmission of Credentials (Sensitive Data Exposure)**  
   - Login credentials transmitted in plaintext.  
   - **Impact:** Credentials exposed via traffic interception.  
   - **Severity:** High (CVSS 7.5)  

3. **Cross-Site Scripting (XSS)**  
   - Search field and input forms reflect unsanitized input.  
   - **Impact:** Cookie theft, session hijacking, malicious script execution.  
   - **Severity:** High (CVSS 7.4 – 8.6)  

4. **Insecure Direct Object Reference (IDOR)**  
   - Basket API allowed manipulation of basket IDs to access other users’ data.  
   - **Impact:** Unauthorized data access & modification.  
   - **Severity:** High (CVSS 8.2)  

5. **Insecure Direct Access & Directory Listing**  
   - `/ftp/` path exposed sensitive files and backups.  
   - **Impact:** Leakage of configs, secrets, backups.  
   - **Severity:** Critical (CVSS 9.0)  

6. **Password Recovery Flaw (Account Takeover)**  
   - Weak security questions allowed password reset using public information.  
   - **Impact:** Full account takeover.  
   - **Severity:** High (CVSS 8.8)  

7. **Other Findings**  
   - Valid account authentication using weak/exposed credentials.  
   - Discovery of hidden administration routes.  
   - Arbitrary file download via path manipulation.  

## 🛡 Recommendations
- Use **parameterized queries (prepared statements)** to prevent SQL injection.  
- Enforce **TLS (HTTPS)** for all communications.  
- Implement **output encoding & CSP** to mitigate XSS.  
- Apply **object-level authorization checks** for APIs.  
- Disable **directory listing** and remove sensitive files from public directories.  
- Replace weak **knowledge-based password recovery** with MFA or token-based resets.  
- Conduct **regular penetration testing** and enable **logging/monitoring** for anomaly detection.  

## 📅 Test Information
- **Target:** https://demo.owasp-juice.shop/  
- **Tester:** Amaan Tamboli  
- **Date:** September 2025  

## 📚 References
- [OWASP Top 10](https://owasp.org/Top10/)  
- [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)  
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)  

---

🚀 **This project demonstrates practical hands-on experience in penetration testing using OWASP Juice Shop.**
