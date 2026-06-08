# Project Name: Insider Threat I

Project Type: Operational Visibility & Purple Team Adversary Emulation

## 📌 Executive Summary
This project demonstrates the design and execution of a localized, multi-VM security operations lab (`SOC Lab`) built on a static `10.10.10.0/24` network. The objective was to emulate an advanced insider threat lifecycle against an Ubuntu endpoint, identify critical telemetry visibility gaps in default operating system logging, and engineer custom kernel-level auditing controls to pipe structured detection analytics into a Splunk SIEM instance.

## 🛠️ Lab Architecture & Topology
The environment consists of a 4-VM architecture isolated inside a private VirtualBox internal network switch (`soc-lab`), preventing external exposure while maintaining internet routing via strategic NAT boundaries:

* **Attacker Node:** Kali Linux (`10.10.10.30`)
* **Target Server:** Ubuntu Linux (`10.10.10.40`)
* **SIEM Platform:** Splunk Enterprise (`10.10.10.10`) 
  * *Note: Port forwarding configured via host loopback (`localhost:8000`) for seamless out-of-band administrative access.*

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | Emulation Tool | Detection Control |
| :--- | :--- | :--- | :--- |
| **Reconnaissance** | Active Scanning (T1595) | Nmap / tcpdump | Network Traffic Flow Analysis |
| **Initial Access** | Valid Accounts (T1078) | Metasploit Auxiliary | Native `auth.log` -> Splunk |
| **Execution** | Command/Script Interpreter (T1059) | Metasploit / Meterpreter | **[Visibility Gap Identified]** |
| **Exfiltration** | Exfil Over Alternative Protocol (T1048) | Python `http.server` | **[Visibility Gap Identified]** |
| **Impact** | Data Destruction (T1485) | Native `rm` utility | **[Visibility Gap Identified]** |

---

## 🚀 Phase 1: Adversary Emulation (Red Team)
Detailed step-by-step technical write-ups of the attack execution, showcasing credential hunting, payload delivery via Meterpreter, and final data sabotage.
👉 [Read the Documentation](docs/01-reconnaissance.md)

## 🛡️ Phase 2: Detection Engineering & Remediation (Blue Team)
Analysis of the out-of-the-box telemetry capabilities, documentation of the unprivileged user visibility gap, and deployment of persistent `execve` auditing rules to restore 100% security monitoring.
👉 [Read the Documentation](docs/04-remediation.md)
