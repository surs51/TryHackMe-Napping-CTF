# THM - Napping

# Tabnabbing -> Root Compromise (Pentest Lab)

This repository documents a full attack chain performed in a controlled CTF environment,
starting from a client-side vulnerability and ending in full root system compromise.

The goal of this assessment was to demonstrate attacker mindset, exploitation logic,
and proper pentest-style reporting.

## Attack Chain Overview

- User-controlled URL reviewed by an authenticated administrator
- Tabnabbing with `window.opener` abuse
- Credential harvesting through a phishing login page
- SSH access using captured credentials
- Privilege escalation via writable scheduled task (Python script)
- Full root compromise via sudo misconfiguration (VIM)

## Skills Demonstrated

- Client-side attack techniques (Tabnabbing)
- Social engineering via UI deception
- Network traffic analysis
- Linux post-exploitation enumeration
- Scheduled task abuse
- Privilege escalation via sudo misconfiguration
- Professional pentest reporting


## Impact

- Administrator account compromise
- Arbitrary code execution
- Full root-level access
- Complete loss of confidentiality, integrity, and availability


## Report

**Full technical report:**  
`report/FullReport.md`
