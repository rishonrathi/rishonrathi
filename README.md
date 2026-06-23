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

* [🔍 OSINT Labs](./OSINT-labs/README.md) - Networking, Google Dorks, Shodan, and Recon Tools.

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
<img width="1210" height="579" alt="sql lab 1" src="https://github.com/user-attachments/assets/19c1dbb4-b193-41a1-acfb-a1632b10390c" />


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

## 🎯 Lab 2: Authentication Bypass (Logging In)

### 🛑 The Problem
The user login portal verification logic is built with dynamic SQL strings. An attacker can manipulate the fields to evaluate the SQL condition to true without providing a valid username or account password.

* **Target Component:** Login Screen Form (`username` and `password` fields)

### 🔬 The Working (Step-by-Step)
1. Located the login form input parameters.
2. Injected a payload into the username field to turn the query logic into an absolute truth, while commenting out the rest of the password evaluation.
3. **Username Payload:** `administrator'--` (or `admin' OR '1'='1`)
4. **Password Field:** Left blank or garbage text.
5. **Resulting Database Query:**
   ```sql
   SELECT * FROM users WHERE username = 'administrator'--' AND password = '...'
   ```
   *The application logged me directly into the `administrator` dashboard because the password constraint was successfully commented out.*

### 📸 Proof of Concept (Screenshot)
<img width="1272" height="601" alt="sql lab 2" src="https://github.com/user-attachments/assets/9bd77cd4-f9cf-4e61-bf93-b49d26c07090" />


### 🛠️ The Solution
The standard fix requires processing credentials strictly as query parameters rather than executable SQL commands. 

```python
# Secure Implementation Example (Python cursor.execute)
query = "SELECT * FROM users WHERE username = %s AND password = %s"
cursor.execute(query, (username_input, password_input))
```

### 💡 What I Learned
* I learned that application access controls are only as secure as the database processing layer below them.
* I realized why using legacy data object handlers without built-in abstraction structures creates severe structural authentication bypass flaws.

---

# Web Vulnerability Lab: Cross-Site Scripting (XSS)

---

## 🎯 Lab 3: Reflected XSS

### 🛑 The Problem
The search query functionality takes user input from the search query parameter and dynamically reflects it back on the results web page without escaping characters or output sanitization.

* **Target Component:** Search input field `/search?q=`

### 🔬 The Working (Step-by-Step)
1. Entered alpha-numeric test strings to check if characters were stripped or encoded.
2. Passed an executable browser payload into the search URL parameters.
3. **Payload Used:** `<script>alert(window.origin)</script>`
4. The web server dynamically built the response page with the raw executable script, forcing the victim's local browser context to execute the alert box popup.

### 📸 Proof of Concept (Screenshot)
<img width="1308" height="552" alt="xss lab 1" src="https://github.com/user-attachments/assets/7bf4930e-fe51-4de2-b706-a1145803ab61" />


### 🛠️ The Solution
Implement **Context-Aware Output Encoding** before rendering variables on the UI, converting dangerous active HTML syntax tags into inert textual entity characters.

```html
<!-- Before (Vulnerable): -->
<p>You searched for: <%= request.getParameter("q") %></p>

<!-- After (Secure Output Encoding): -->
<p>You searched for: <%= org.owasp.encoder.Encode.forHtml(request.getParameter("q")) %></p>
<!-- Result renders safely textually as: &lt;script&gt;alert(1)&lt;/script&gt; -->
```

### 💡 What I Learned
* I learned that Reflected XSS requires social engineering (like a phishing link click) because the malicious payload is not hosted on the remote server itself.
* I learned how to read DOM source files using web developer inspection tabs to trace where input flags are reflected.

---

## 🎯 Lab 4: Stored XSS

### 🛑 The Problem
The comment module allows site visitors to submit feedback that gets permanently written to the server's backend database repository. Because the feedback display logic fails to clean strings, the saved exploit runs on the machine of *any* future user viewing the comment board.

* **Target Component:** Public Blog Comment Section

### 🔬 The Working (Step-by-Step)
1. Submitted a benign comment to verify it gets stored and loaded upon refresh.
2. Injected a hidden script block masquerading inside an image element handler payload.
3. **Payload Used:** `<img src=x onerror=alert(document.cookie)>`
4. When any regular visitor or administrator browses to this specific blog entry, the broken image tag forces an execution error, automatically extracting session tokens.

### 📸 Proof of Concept (Screenshot)
<img width="1215" height="553" alt="xss lab 2" src="https://github.com/user-attachments/assets/15b5736b-53d3-41b7-b05a-72c997dd100a" />


### 🛠️ The Solution
Apply both server-side **Input HTML Sanitization** (using robust libraries like DOMPurify) and strict **Content Security Policies (CSP)** headers to stop unauthorized script sources.

```javascript
// Secure JavaScript Sanitization Example
const DOMPurify = require('dompurify');
const cleanComment = DOMPurify.sanitize(userInputComment);
```

### 💡 What I Learned
* I discovered that Stored XSS is exponentially more high-risk than Reflected XSS because it operates silently and scales across multiple victim accounts without direct interaction.
* I realized the value of restricting browser scripts using security policies like the `HttpOnly` cookie flag to mitigate session hijacking via script manipulation. 

## 📂 Active Repositories & Evidence of Work

### 1. [Cybersecurity-Concepts-and-Protocols](https://github.com/rishonrathi/cybersecurity-foundations)
- **Scope:** Detailed research and breakdown of essential networking protocols and security models.
- **Key Focus:** Analyzing how data travels securely across networks.

### 2. [TryHackMe-Lab-Writeups](https://github.com/rishonrathi/tryhackme-lab-logs)
- **Scope:** Verifiable step-by-step documentation of completed hands-on beginner labs.
- **Key Focus:** Linux Fundamentals and basic network scanning tools.
