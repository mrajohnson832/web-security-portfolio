# Web Application Security Portfolio

I'm **Anthony Johnson**, a web application penetration tester focused on **business-logic
and access-control flaws**, clean **findings reporting**, and building my own tooling. This
repository is a working portfolio of my methodology and projects.

**Focus:** web/API penetration testing 
- access control & broken authorization 
- authentication & session flaws 
- business-logic vulnerabilities 
- recon & attack-surface mapping

**Certifications:** 
- CompTIA PenTest+ 
- *(in progress: Burp Suite Certified Practitioner)*

---

## What's inside

### 🔎 Methodology write-ups
Technical write-ups from the **PortSwigger Web Security Academy** (deliberately-vulnerable
training targets). These aren't answer keys, each one documents how I approached the problem,
the non-obvious insight, and a **reusable testing rule** I took away from it. They show how I
*think* about finding bugs, not just that I completed a lab.

- [Account Takeover via Weak Isolation on a Dual-Use Endpoint](writeups/account-takeover-dual-use-endpoint.md)
- [Account Takeover by Chaining Stored XSS, Cookie Theft, and Offline Password Cracking](writeups/account-takeover-xss-to-offline-cracking.md)
- [Privilege Escalation via a Flawed Login State Machine](writeups/privilege-escalation-flawed-login-state-machine.md)
- *(more added as I write them up)*

**Topics covered to date:** 
- Access Control / Broken Authorization (full topic)
- Authentication (full topic)
- Business Logic (in progress)

### 🛠️ Tooling: `ajsecx`, a pentest assistant (design case study)
I designed and directed a local-first, **RAG-grounded pentest assistant** (FastAPI, SQLite,
local LLM, file-based retrieval) that guides engagements through curated playbooks, orchestrates
CLI tools, and drafts findings reports. A deliberate security-design choice: the LLM is scoped to
**source-grounded advisory and report-drafting only, never the bug-finder**, to prevent
vulnerability hallucination.
→ [ajsecx (design case study)](https://github.com/mrajohnson832/ajsecx). *Source kept private;
the repo documents the design and reasoning, not the code.*

### 🧪 Home lab: infrastructure hardening *(in progress)*
A small-business Windows domain lab (Active Directory, hardening to a CIS baseline, SIEM
detections, backups) that I build, harden, attack, and detect, demonstrating defensive work
alongside the offensive.
→ [windows-domain-lab](https://github.com/mrajohnson832/windows-domain-lab).

---

## How to read this
Everything here targets **deliberately-vulnerable labs or my own home lab**, never a real or
third-party system. Handling scope and sensitive data responsibly is part of the job, so you
won't find anything in here that shouldn't be public.

## Contact
[LinkedIn](https://www.linkedin.com/in/anthony-johnson-767813256/) | [Credly](https://www.credly.com/users/ajohnson832/badges/credly) | [Email](mailto:mr.ajohnson832@gmail.com)
