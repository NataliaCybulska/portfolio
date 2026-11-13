## Hello!<br>
**If you're here, you're possibly considering me for a position in your company.**<br>
**Thank you for your time and for giving me a chance!**

My background is QA - since then I've been leaning hard into infrastructure, secure coding practices, hands-on offensive and defensive security. This repository is the evidence trail: certifications, exploitation write-ups, projects and real code I've shipped.

---

## Certifications

CompTIA Security+, AWS Certified Cloud Practitioner, Microsoft Certified: Security, Compliance, and Identity Fundamentals, and the Google Cybersecurity Professional Certificate. I've also been living on TryHackMe: top 1% rank, 341 rooms completed, 308-day streak. Full breakdown, dates, and verification details in [certifications](certifications/README.md).

---

## Security Write-ups

Nine write-ups from hands-on TryHackMe rooms, from breaking things to catching them:

- Application: breaking a web app found through static code analysis 
- Infrastructure: insecure IaC to host root, and two Kubernetes clusters taken to cluster-admin 
- AI: jailbreaking chatbots, investigating a backdoored ML model, and hardening a leaky RAG assistant 
- Defense: climbing the Pyramid of Pain to detect malware, and tracing a credential-phishing campaign from inbox to kit

Each one keeps the dead ends in and maps findings to the relevant framework, from the OWASP Top 10 to MITRE ATT&CK. The Payload room hit close to home after working on a model server: it’s about what happens when the model you’re serving is the attack. [security-writeups](security-writeups/README.md) has the full walkthroughs.

---

## Projects

**[Twin Bank Security Lab](https://github.com/NataliaCybulska/twin-bank-security-lab)** 

A web security lab I built from scratch: one TypeScript bank running twice from the same image, once vulnerable, once hardened, with an attacker site firing the same attacks at both.

- **Findings:** stored XSS, CORS misconfiguration, CSRF and clickjacking, each traced from reproduction to the exact line, then to a fix the hardened build enforces.
- **App hardening:** output encoding, nonce-based CSP, HttpOnly and SameSite cookies, anti-CSRF tokens and exact-match CORS, each pinned to the finding it closes.
- **Lab hardening:** containers run non-root with read-only filesystems, dropped Linux capabilities and CPU/memory/process limits; ports bound to localhost; the Docker socket left unmounted; base images pinned by digest and dependencies verified against lockfile hashes.
- **Scanning:** Semgrep for static analysis (SAST), OWASP ZAP against the running app (DAST), Google CSP Evaluator for the CSP, and Trivy and npm audit for dependencies. 

![Clickjacking the vulnerable bank: the slider reveals the hidden frame over the real Confirm transfer button](https://raw.githubusercontent.com/NataliaCybulska/twin-bank-security-lab/main/images/clickjacking-demo.gif)

---

## Engineering Samples

Real, merged contributions to [OpenVINO™ Model Server](https://github.com/openvinotoolkit/model_server), Intel's open-source model-serving project. I refactored legacy test infrastructure without breaking the suite while I did it. Most of what I've shipped professionally sits behind an NDA, so [engineering-samples](engineering-samples/README.md) is the exception.

---

## Academic Work

My engineering thesis: generating impressionist paintings with StyleGAN2, written in Polish. [academic-work](academic-work/README.md) has more dog photos and bad puns than a thesis strictly requires.

---

Thank you! If you have any questions, I will be happy to answer them.
