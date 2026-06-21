## 🏆 Verified Credentials & Certifications
*A record of structured training completed through recognized academic and industry bodies.*

- **Google Cybersecurity Professional Certificate**  
  *Issued by:* Google via Coursera (2026)  
  *Key Areas:* Network Security (TCP/IP), Linux Terminal Operations, Python Automation Baselines, SIEM Tools, and Packet Analysis.  
  *Verification:* [https://www.coursera.org/account/accomplishments/certificate/NGIEODMIDB4S]

- **Cybersecurity Foundational Training**  
  *Issued by:* Tech Mahindra Foundation × National Association of Software and Services Companies (NASSCOM)  
  *Platform:* Skill India Hub  
  *Key Areas:* Industry-aligned security fundamentals, corporate compliance basics.  
  *Verification:* [https://drive.google.com/file/d/1IltW6g2iRLe7czfjpW2gwKAOuVdUkBVT/view?usp=drive_link]
                  [https://drive.google.com/file/d/1VTXEigEZABDen3NuXdg8t90ywg8IezjY/view?usp=drive_link]

- **Cybersecurity Fundamentals / Specialized Course**  
  *Issued by:* Great Learning Academy  
  *Key Areas:* Core security concepts, network defense baselines, threat overview.  
  *Verification:* [https://drive.google.com/file/d/1Yp4uTmzeZKqhLrazsA2a6-OhQu_-rpxn/view?usp=drive_link]

  # Web Vulnerability Lab: SQL Injection (SQLi)

## ⚠️ Educational Disclaimer
All labs were conducted strictly within a controlled, authorized environment for educational and defensive learning purposes.

---

## 🎯 Lab 1: Retrieving Hidden Data

### 🛑 The Problem
The web application features a product filter page where a category query parameters is passed directly into a backend database query without sanitization. This allows unauthenticated users to manipulate the query logic and view unlisted, hidden inventory items.

* **Target URL/Parameter:** `/filter?category=Gifts`
* **Vulnerable Backend Code Concept:** 
  ```sql
  SELECT * FROM products WHERE category = 'Gifts' AND released = 1
  ```

### 🔬 The Working (Step-by-Step)
1. Appended a single quote `'` to the category name to test for database error syntax.
2. Injected a boolean condition payload into the parameter to break the logic boundary.
3. **Payload Used:** `' OR 1=1--`
4. **Resulting Database Query:**
   ```sql
   SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
   ```
   *The `--` sequence commented out the remainder of the query, forcing the database to evaluate `1=1` (always true) and display every single item in the table.*

### 📸 Proof of Concept (Screenshot)
![SQL Lab 1](sql_lab_1.png)

### 🛠️ The Solution
To remediate this issue, developers must stop concatenating user string inputs directly into SQL commands. Implement **Parameterized Queries (Prepared Statements)**:

```php
// Secure Implementation Example (PHP PDO)
$stmt = $pdo->prepare('SELECT * FROM products WHERE category = :category AND released = 1');
$stmt->execute(['category' => $userInput]);
$products = $stmt->fetchAll();
```

### 💡 What I Learned
* I learned that client-side input safety checks can be easily bypassed using tools like Burp Suite or direct URL manipulation.
* I understood how database comment symbols (`--`, `#`, or `/*`) change logic branches mid-execution by dropping native queries.

---

## 📂 Active Repositories & Evidence of Work

### 1. [Cybersecurity-Concepts-and-Protocols](https://github.com/rishonrathi/cybersecurity-foundations)
- **Scope:** Detailed research and breakdown of essential networking protocols and security models.
- **Key Focus:** Analyzing how data travels securely across networks.

### 2. [TryHackMe-Lab-Writeups](https://github.com/rishonrathi/tryhackme-lab-logs)
- **Scope:** Verifiable step-by-step documentation of completed hands-on beginner labs.
- **Key Focus:** Linux Fundamentals and basic network scanning tools.
