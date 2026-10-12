# Security Write-ups

[TryHackMe](https://tryhackme.com) is a hands-on platform for practising security skills. Each "room" is a self-contained, deliberately vulnerable environment with hidden flags you find by actually exploiting it. Similar idea to a CTF (capture the flag), with a guided difficulty curve.

These are write-ups from rooms I completed, grouped by layer: the application code, the infrastructure underneath it, and the AI sitting on top. Turns out breaking things has layers, and so does cleaning up after it. Each write-up keeps the dead ends in and explains why every technique works. 

## Application

| Write-up | What it covers | Key techniques | OWASP |
| --- | --- | --- | --- |
| [Mother's Secret](mothers-secret.md) ([room](https://tryhackme.com/room/codeanalysis)) | Finding a path traversal through static analysis, then exploiting it on the live app | SAST (ESLint, Semgrep), manual code review, path traversal (Burp Suite), abusing the auth flow | [A01:2025](https://top10.owasp.org/2025/A01_2025-Broken_Access_Control/) Broken Access Control |

## Infrastructure

| Write-up | What it covers | Key techniques | OWASP |
| --- | --- | --- | --- |
| [On-Premises IaC](on-premises-iac.md) ([room](https://tryhackme.com/room/onpremisesiac)) | Reading Vagrant and Ansible files, then going from a web app to root on the host | IaC review, port scanning (Nmap), OS command injection (Burp Suite), SSH tunnelling, synced-folder pivot, Docker group privilege escalation | [A05:2025](https://top10.owasp.org/2025/A05_2025-Injection/) Injection, [A02:2025](https://top10.owasp.org/2025/A02_2025-Security_Misconfiguration/) Security Misconfiguration |

## AI

| Write-up | What it covers | Key techniques | OWASP |
| --- | --- | --- | --- |
| [Prompt Injection & Jailbreaks](ai-prompt-injection-jailbreaks.md) (rooms: [Jailbreaking](https://tryhackme.com/room/jailbreaking), [LLMborghini](https://tryhackme.com/room/llmborghini), [White Rabbit](https://tryhackme.com/room/whiterabbit)) | Talking three LLM chatbots and agents into leaking secrets | Persona jailbreak, pretexting, task reframing, refusal suppression | [LLM01](https://genai.owasp.org/llmrisk/llm01-prompt-injection/), [LLM02](https://genai.owasp.org/llmrisk/llm022025-sensitive-information-disclosure/), [LLM09](https://genai.owasp.org/llmrisk/llm092025-misinformation/) |
| [Payload](ai-supply-chain-payload.md) ([room](https://tryhackme.com/room/payload)) | Investigating a malicious model in production and its tampered replacement | Log analysis, pickle decompilation (fickling), model scanning (modelscan), dependency audit (pip-audit), typosquat spotting, Keras model inspection (h5py) | [LLM03](https://genai.owasp.org/llmrisk/llm032025-supply-chain/), [LLM04](https://genai.owasp.org/llmrisk/llm042025-data-and-model-poisoning/) |

Application and infrastructure rows map to the [OWASP Top 10:2025](https://top10.owasp.org/2025/), AI rows to the [OWASP Top 10 for LLM Applications 2025](https://genai.owasp.org/llm-top-10/).
