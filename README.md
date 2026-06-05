# Detection Engineering CI/CD Platform

> An automated detection validation pipeline that simulates real MITRE ATT&CK techniques against a live Windows 11 endpoint, validates custom Wazuh detection rules, and generates professional SOC coverage reports — end to end in a single command.

![Coverage](https://img.shields.io/badge/ATT%26CK%20Coverage-100%25-brightgreen)
![Techniques](https://img.shields.io/badge/Techniques-5%2F5-brightgreen)
![SIEM](https://img.shields.io/badge/SIEM-Wazuh%204.7.5-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%2011-blue)
![AI](https://img.shields.io/badge/AI-Claude%20API-purple)

---

## Overview

Most detection engineering work happens in silos — someone writes a rule, deploys it, and hopes it works. This project solves that by building a **CI/CD pipeline for detections**: every rule is automatically tested against a real attack simulation and must pass before it's considered valid coverage.

This mirrors how mature security teams at enterprise organizations validate their detection content — the difference is this was built from scratch as a solo project.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     ATTACK SIMULATION LAYER                     │
│         Atomic Red Team → Windows 11 Host (ATT&CK Techniques)  │
└─────────────────────────┬───────────────────────────────────────┘
                          │ Sysmon Telemetry
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                        DETECTION LAYER                          │
│     Sysmon → Wazuh Agent → Wazuh Manager → Custom Rules        │
└─────────────────────────┬───────────────────────────────────────┘
                          │ Alerts + Rule Firing
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                       VALIDATION LAYER                          │
│   Python Pipeline → Wazuh Indexer API → PASS/FAIL per Technique│
└─────────────────────────┬───────────────────────────────────────┘
                          │ Coverage Data
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                       REPORTING LAYER                           │
│        HTML Dashboard + Claude AI SOC Incident Report          │
└─────────────────────────────────────────────────────────────────┘
```

---

## ATT&CK Coverage Results

| Technique | Name | Tactic | Custom Rule | Status |
|-----------|------|--------|-------------|--------|
| T1057 | Process Discovery | Discovery | 100002 | ✅ PASS |
| T1059.001 | PowerShell Execution | Execution | 100004 | ✅ PASS |
| T1082 | System Information Discovery | Discovery | 100006 | ✅ PASS |
| T1053.005 | Scheduled Task Persistence | Persistence | 100003 | ✅ PASS |
| T1027 | Base64 Defense Evasion | Defense Evasion | 100007 | ✅ PASS |

**5/5 techniques detected — 100% coverage**

---

## Features

- **Automated attack simulation** via Atomic Red Team on a live Windows 11 endpoint
- **Deep telemetry** via Sysmon with SwiftOnSecurity configuration
- **Custom detection rules** authored in Wazuh XML, mapped to MITRE ATT&CK
- **CI/CD validation pipeline** that queries the Wazuh indexer API and scores each rule PASS/FAIL
- **HTML coverage dashboard** with ATT&CK technique breakdown and coverage ring
- **AI-generated SOC incident report** via Claude API — professional markdown output
- **Single command execution** — entire pipeline runs with `bash pipeline/run_pipeline.sh`

---

## Stack

| Component | Technology |
|-----------|-----------|
| SIEM | Wazuh 4.7.5 (Docker) |
| Endpoint Telemetry | Sysmon + SwiftOnSecurity config |
| Attack Simulation | Atomic Red Team (invoke-atomicredteam) |
| Target Endpoint | Windows 11 |
| Detection Rules | Custom Wazuh XML rules |
| Pipeline | Python 3 |
| AI Reporting | Claude API (Anthropic) |
| Version Control | Git + GitHub |

---

## Project Structure

```
detection-engineering-cicd/
├── pipeline/
│   ├── validate_detections.py   # Core CI/CD validation engine
│   ├── generate_dashboard.py    # HTML ATT&CK coverage dashboard
│   ├── generate_report.py       # AI-powered SOC incident report
│   └── run_pipeline.sh          # Master pipeline runner
├── detection-rules/
│   └── local_rules.xml          # Custom Wazuh detection rules
├── reports/                     # Generated coverage reports (JSON/HTML/MD)
├── attack-simulations/          # ATT&CK technique documentation
└── docs/                        # Architecture and setup documentation
```

---

## Usage

```bash
# Set your Anthropic API key
export ANTHROPIC_API_KEY=your-key-here

# Run the full pipeline (validate + dashboard + AI report)
bash pipeline/run_pipeline.sh

# Or run individual components
python3 pipeline/validate_detections.py   # Validate all detection rules
python3 pipeline/generate_dashboard.py    # Generate HTML dashboard
python3 pipeline/generate_report.py       # Generate AI incident report
```

### Pipeline Output

```
============================================================
  Detection Engineering CI/CD Pipeline
  Run time: 2026-06-04 18:39:01
============================================================
[+] Wazuh API authenticated successfully

[*] Testing T1059.001 - PowerShell Execution
    [✓] PASS - Rule 100004 confirmed in alerts!

[*] Testing T1082 - System Information Discovery
    [✓] PASS - Rule 100006 confirmed in alerts!

[*] Testing T1057 - Process Discovery
    [✓] PASS - Rule 100002 confirmed in alerts!

[*] Testing T1053.005 - Scheduled Task Persistence
    [✓] PASS - Rule 100003 confirmed in alerts!

[*] Testing T1027 - Base64 Defense Evasion
    [✓] PASS - Rule 100007 confirmed in alerts!

============================================================
  DETECTION COVERAGE REPORT
  Total: 5 | Passed: 5 | Failed: 0 | Coverage: 100.0%
============================================================
```

---

## Challenges & Problem Solving

This section documents the real technical obstacles encountered and how they were resolved — because detection engineering is mostly debugging.

---

### Challenge 1: Wazuh Not Supported on Arch Linux

**Problem:** Wazuh's official install script (`wazuh-install.sh -a`) immediately exited with:
```
ERROR: Couldn't find type of system
```
Arch Linux is not in Wazuh's supported distro list and the script has no fallback.

**Solution:** Deployed Wazuh entirely via Docker using the official `wazuh-docker` single-node compose stack. This actually produced a more portable and reproducible environment than a native install would have — a better outcome than the original plan.

**Lesson:** When a tool doesn't support your OS, containerization is often a cleaner solution than fighting the installer.

---

### Challenge 2: Disk Space Exhaustion Mid-Project

**Problem:** During PowerShell installation, the root partition hit 100% capacity:
```
Your Root partition is running out of disk space; 0 MiB remaining (0%)
```
The VM had 49GB allocated but Docker image layers from multiple previous projects had consumed nearly all of it.

**Solution:** Ran `docker system prune -a --volumes -f` which safely removed unused images and volumes from previous projects, recovering 4.7GB. Also cleared journal logs and build caches to recover additional space.

**Lesson:** Docker image sprawl is a real operational concern. Regular pruning and monitoring disk usage should be part of any lab hygiene practice.

---

### Challenge 3: Custom Detection Rules Not Firing

**Problem:** After writing 6 custom Wazuh rules using `if_sid` to chain from parent rules (92027, 92031, 92034), the validation pipeline showed 0% coverage — none of the custom rules were appearing in alerts even though the parent rules were firing correctly.

**Root cause investigation:**
1. First checked if parent rules were firing — they were (hundreds of hits)
2. Checked if rule syntax was valid — it was
3. Queried the raw alerts log and found tasklist.exe and wmic.exe events were triggering **different** Sysmon event IDs than expected
4. Discovered Wazuh's rule chaining behavior: `if_sid` on a rule that is itself a child rule (like 92041) requires `if_matched_sid` instead

**Solution:** Changed the base `if_sid` references to `61603` (the raw Sysmon process creation event) for process-based rules, and changed `if_sid` to `if_matched_sid` for the T1027 Base64 registry rule that chains from 92041. Coverage went from 0% → 100%.

**Lesson:** Understanding the difference between `if_sid` and `if_matched_sid` in Wazuh's rule chaining model is critical for detection engineers working on custom rule development.

---

### Challenge 4: Wazuh API Endpoint Mismatch

**Problem:** The initial pipeline used `/siem/alerts` endpoint which returned 404. All 5 techniques showed FAIL despite rules clearly firing in the dashboard.

**Solution:** Switched to querying the Wazuh OpenSearch indexer directly at `https://localhost:9200/wazuh-alerts-*/_search` using the OpenSearch DSL query format. This gave direct access to the alerts index and returned accurate results.

**Lesson:** Always verify API endpoint availability against the specific version you're running. Documentation for older versions can be misleading.

---

### Challenge 5: API Key Exposed in Git Commit

**Problem:** GitHub's push protection blocked the push because the Anthropic API key was hardcoded in `generate_report.py` and detected in the commit history.

**Solution:**
1. Replaced hardcoded key with `os.environ.get("ANTHROPIC_API_KEY")`
2. Created `.env` file for local use and added it to `.gitignore`
3. Used `git filter-branch` to rewrite history and remove the exposed secret
4. Force pushed the clean history

**Lesson:** Never hardcode secrets in source files. Use environment variables from the start. GitHub's secret scanning is a valuable safety net but the correct fix is never committing secrets in the first place.

---

## Key Takeaways for Interviews

**"What was the hardest part of this project?"**
> The rule chaining issue — debugging why custom rules weren't firing despite parent rules working correctly required understanding Wazuh's internal rule evaluation model at a deep level. The fix was a single XML attribute change (`if_sid` → `if_matched_sid`) but finding it required systematic log analysis and reading source documentation carefully.

**"How does this relate to real-world detection engineering?"**
> Real detection engineering teams face the same core problem: rules get written and deployed but nobody validates they actually work against the techniques they're supposed to detect. This pipeline automates that validation — the same concept used by mature security teams running detection-as-code workflows.

**"Why did you choose this stack?"**
> Wazuh because it's open source and widely used in enterprise environments. Sysmon because it's the gold standard for Windows endpoint telemetry. Atomic Red Team because it provides standardized, reproducible ATT&CK technique simulations. The combination mirrors what real blue teams use.

---

## Future Improvements

- [ ] Add GitHub Actions workflow to run pipeline on every rule change
- [ ] Expand to 20+ ATT&CK techniques across more tactics
- [ ] Add Sigma rule conversion for multi-SIEM support
- [ ] Integrate Splunk as second SIEM target
- [ ] Add detection drift alerting — notify when a previously passing rule starts failing
- [ ] Build ATT&CK Navigator heatmap export

---

## Author

**Fernando** — AAS Information Assurance  
Focused on detection engineering, SOC automation, and AI-assisted security operations.  
GitHub: [@cpt-ferna02](https://github.com/cpt-ferna02)
