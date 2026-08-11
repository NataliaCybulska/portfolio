## Security Writeups

[TryHackMe](https://tryhackme.com) is a hands-on platform for practicing security skills - each "room" is a self-contained, deliberately vulnerable environment with hidden flags you find by actually exploiting it, not just reading about it. Similar idea to a CTF (capture-the-flag), with a guided difficulty curve.

These are writeups from two rooms I completed - one lives in application code, the other in the infrastructure underneath it. Turns out breaking things has layers.

| Writeup | Focus | Techniques |
|---|---|---|
| [Mother's Secret](mothers-secret.md) | Exploiting a vulnerable web app found via static code analysis | SAST tooling (ESLint, Semgrep), manual code review, Burp Suite, path traversal, crafted HTTP requests |
| [On-Premises IaC](on-premises-iac.md) | Exploiting insecure Infrastructure-as-Code | Vagrant/Ansible static analysis, Nmap, Burp Suite, OS command injection (RCE), SSH tunneling, synced-folder host pivoting, Docker group privilege escalation |
