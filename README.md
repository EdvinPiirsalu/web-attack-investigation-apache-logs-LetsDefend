# web-attack-investigation-apache-logs-LetsDefend
SOC Analyst investigation of a web server compromise using Apache access logs (LetsDefend)


# Web Attack Investigation – Apache Access Log Analysis

**Platform:** LetsDefend **Log Source:** Apache `access.log`
**Methodology:** Manual log analysis, mapped to the Cyber Kill Chain

## Overview

This investigation analyzes a compromised web server using Apache access logs. The objective was to identify the attacker's actions across each stage of the attack and map them to the Cyber Kill Chain framework.



### 1. Reconnaissance — Automated Scanning Tool

**Question:** Which automated scan tool did attacker use for web reconnaissance?

Multiple requests were identified originating from the same source IP within a very short time window. Inspection of the HTTP headers revealed `Nikto` in the User Agent field.

**Finding:** The attacker used **Nikto**, a web vulnerability scanner, to perform reconnaissance against the target.

**Evidence**
- User-Agent field contains `Nikto`
- High volume of automated requests
- Requests generated within seconds of each other

  <img width="2492" height="1202" alt="1" src="https://github.com/user-attachments/assets/d6d7a0e7-4e16-4ef6-bbb2-0676edfa4841" />


  <img width="1627" height="245" alt="2" src="https://github.com/user-attachments/assets/54c8112e-9cfd-4b0e-b9e4-38a13f69e580" />



### 2. Discovery — Directory Brute Force

**Question:** After web reconnaissance activity, which technique did attacker use for directory listing discovery?

Following reconnaissance, the attacker requested a large number of directories and files. Most returned `404 Not Found`, with some returning `200 OK` for valid resources. Apprears testing with list of words in alphabetical order.

**Finding:** The attacker performed a **Directory Brute Force Attack**

**Evidence**
- Large number of requests
- Multiple `404` responses followed by successful `200` responses
- Systematic request pattern consistent with list of words

  <img width="2547" height="1172" alt="3" src="https://github.com/user-attachments/assets/844e2be8-6987-45ea-8124-28ddcc0b040a" />


  <img width="1640" height="246" alt="4" src="https://github.com/user-attachments/assets/ce6fb969-f0c0-43c9-87b7-e6de90a67084" />



### 3. Credential Attack — Login Brute Force

**Question:** What is the third attack type after directory listing discovery?

After discovering `login.php`, the attacker shifted focus to the authentication mechanism, repeatedly submitting different credential combinations.

**Finding:** The attacker conducted a **Brute Force Login Attack** against the web application's authentication portal.

**Evidence**
- Repeated requests to `login.php`
- Consistent request pattern
- Lots of authentication attempts within a short period

  <img width="2527" height="1182" alt="5" src="https://github.com/user-attachments/assets/5ffc5e37-d565-4e14-9142-d6172e9f922e" />


  <img width="1642" height="246" alt="6" src="https://github.com/user-attachments/assets/45dd7b40-ff57-41fe-944b-30afa51d60e6" />



### 4. Authentication Bypass

**Question:** Is the third attack successful?

Initial requests to `login.php` returned `HTTP 200` with similar response sizes, indicating failed login attempts. Later in the log, an `HTTP 302` redirect appears, leading to `portal.php` followed by successful access to other pages. Response size increased.

**Finding:** **Yes** — the brute force attack was successful.

**Evidence**
- Status code change from `200` → `302`
- Redirect to authenticated resource (`portal.php`)
- Increased response size after authentication
- Access to internal application pages

  <img width="2505" height="1177" alt="7" src="https://github.com/user-attachments/assets/9821a7d0-206c-46c1-b0ec-8ed53257a46e" />


  <img width="1630" height="245" alt="8" src="https://github.com/user-attachments/assets/02b9df38-7fad-4779-a7c9-f2b860cc98f3" />




### 5. Execution — Command Injection

**Question:** What is the name of fourth attack?

After gaining access, the attacker began supplying OS commands through application parameters, exploiting a vulnerability allowing system command execution via `phpi.php`.

**Finding:** The attacker performed **Command Injection**.

**Evidence**
- Requests containing operating system commands
- Commands executed through application parameters
- `phpi.php` used as the attack vector

  <img width="1627" height="246" alt="10" src="https://github.com/user-attachments/assets/178c5c10-dcfc-4b62-a00c-2ae0e383111b" />


### 6. First Command Injection Payload

**Question:** What is the first payload for 4th attack?

```
whoami
```

A standard reconnaissance command used by attackers to confirm the identity and privileges of the user running the web application.


<img width="2550" height="1177" alt="11 1" src="https://github.com/user-attachments/assets/ced437ef-69e3-416c-8c32-6a8695615f46" />


<img width="1632" height="247" alt="11" src="https://github.com/user-attachments/assets/ba851ba2-1ce2-4110-8d46-cacbe8b6d20f" />



### 7. Persistence — Malicious User Creation

**Question:** Is there any persistency clue for the victim machine in the log file ? If yes, what is the related payload?


Further review revealed commands to create a new local user account:

```
useradd hacker
```

followed by commands to set the account password:

```
Asd123!!
```

**Finding:** **Yes**  persistence was established by creating a new local user account, allowing the attacker to regain access after the initial compromise.


<img width="2511" height="1200" alt="12" src="https://github.com/user-attachments/assets/359c8429-6cc9-459d-bee4-60584f288ca1" />


<img width="1627" height="242" alt="13" src="https://github.com/user-attachments/assets/c264916d-ed47-468c-afd6-7a741c623106" />



## Conclusion

Analysis of the Apache access logs revealed a multi-stage attack chain:

1. Reconnaissance using Nikto
2. Directory brute forcing
3. Credential brute force attack
4. Successful authentication bypass
5. Command injection exploitation
6. Persistence via malicious user account creation

This investigation demonstrates how web server logs alone can be used to make a complete attack timeline and identify techniques of multiple stages of the Cyber Kill Chain. It shows the importance of monitoring authentication events, abnormal HTTP request patterns, and command execution attempts to detect and respond to web-based attacks.
