# Insider-Threat I

**Project Type:** Operational Visibility & Purple Team Adversary Emulation

**Screenshots:** See `/screenshots/` folder for screenshots of every stage of the project from attack to detection

---

## 📌 Executive Summary

This project demonstrates the design and execution of a localized, multi-VM security operations lab (**SOC Lab**) built on a static `10.10.10.0/24` network. The objective was to emulate an advanced insider threat lifecycle against an Ubuntu endpoint, identify critical telemetry visibility gaps in default operating system logging, and engineer custom kernel-level auditing controls to pipe structured detection analytics into a Splunk SIEM instance.

---

## 🛠️ Lab Architecture & Topology

The environment consists of a 3-VM architecture isolated inside a private VirtualBox internal network switch (`soc-lab`), preventing external exposure while maintaining internet routing via strategic NAT boundaries.

### Systems

| Role | Host | IP Address |
|--------|--------|--------|
| Attacker Node | Kali Linux | `10.10.10.30` |
| Target Server | Ubuntu Linux | `10.10.10.40` |
| SIEM Platform | Splunk Enterprise | `10.10.10.10` |

> **Note:** Port forwarding was configured through host loopback (`localhost:8000`) to provide administrative access to Splunk from the physical host browser.

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | Emulation Tool | Detection Control |
|----------|----------|----------|----------|
| Reconnaissance | Active Scanning (T1595) | Nmap / tcpdump | Network Traffic Flow Analysis |
| Initial Access | Valid Accounts (T1078) | Metasploit Auxiliary | Native `auth.log` → Splunk |
| Execution | Command and Scripting Interpreter (T1059) | Metasploit / Meterpreter | **Visibility Gap Identified** |
| Exfiltration | Exfiltration Over Alternative Protocol (T1048) | Python `http.server` | **Visibility Gap Identified** |
| Impact | Data Destruction (T1485) | Native `rm` Utility | **Visibility Gap Identified** |

---

# 🚀 Phase 1: Adversary Emulation (Red Team Execution)

## 1. Reconnaissance & Enumeration

The adversary initiated the attack lifecycle by performing active infrastructure mapping against the target subnet to identify exposed service vectors.

```bash
nmap -sV -p- 10.10.10.40
```

This active network profiling generated high-volume traffic anomalies over port 22, which were verified locally using packet inspection utilities such as `tcpdump`.

---

## 2. Initial Access via Brute Force

Leveraging automated credential-spraying techniques through Metasploit auxiliary modules, the attacker targeted exposed authentication services to harvest valid credentials belonging to the unprivileged user account (`richard`).

```msfconsole
msf> auxilary/scanner/ssh/ssh_login
```

Upon locating a valid password match, session state verification confirmed successful Initial Access.

---

## 3. Post-Exploitation, Exfiltration & Sabotage

The interactive shell was upgraded to a more stable Meterpreter session and hardened using a Python pseudo-terminal:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

The adversary identified high-value business assets within the user's home directory and initiated exfiltration by launching an ad hoc Python HTTP server:

```bash
python3 -m http.server 8000
```

This enabled cleartext file transfers that could be observed directly on the network.

To complete the attack lifecycle and simulate operational impact, the adversary executed destructive commands using the native `rm` utility to remove targeted assets and configuration files.

---

# 🛡️ Phase 2: Detection Engineering & SIEM Integration (Blue Team)

## 🔍 The Telemetry Blind Spot

During post-incident review, despite an active Splunk Universal Forwarder ingesting events from:

```text
/var/log/audit/audit.log
```

the post-exploitation commands executed by `richard` (e.g., `rm`, `python3`) generated no meaningful telemetry within Splunk.

By default, Linux audit configurations prioritize:

- Authentication events
- Privilege escalation activities (`sudo`)
- Administrative account monitoring

As a result, standard low-privileged process executions frequently occur without visibility.

An insider threat operating with valid credentials could therefore:

- Access sensitive data
- Exfiltrate information
- Destroy assets

while remaining effectively invisible to the security operations team.

---

## 🛠️ Persistent Kernel Auditing Rules

To close this visibility gap without overwhelming Splunk's free-tier licensing limits through indiscriminate process logging, a targeted auditing strategy was implemented.

Persistent audit rules were written to intercept the kernel-level `execve` syscall for specific user IDs:

```text
-a always,exit -F arch=b64 -F uid=1002 -S execve -k v_user_commands
-a always,exit -F arch=b64 -F uid=1000 -S execve -k v_user_commands
```

> **Note:** richard is the user with uid 1002

These rules were stored within:

```text
/etc/audit/rules.d/audit.rules
```

The audit daemon was then restarted to load the new configuration into kernel memory:

```bash
sudo service auditd restart
```

---

## 📊 The Insider Threat Monitoring Desk

With targeted kernel telemetry successfully streaming into Splunk, audit events were ingested, parsed, and normalized into an analyst-facing dashboard.

The resulting monitoring solution provides:

- Real-time process visibility
- Insider threat monitoring
- Behavioral anomaly detection
- Unauthorized shell activity identification
- Enhanced investigative context during incident response

---

# 🧠 Lessons Learned & Engineering Reflections

## The Architectural Need for EDR

Native operating system auditing frameworks such as Linux `auditd` were designed primarily for compliance and accountability—not modern threat hunting.

Process execution events are fragmented across multiple records, including:

- `type=SYSCALL`
- `type=EXECVE`

Command arguments are often represented as raw hexadecimal strings, requiring significant parsing effort before they become analyst-friendly.

This highlights the value of Endpoint Detection and Response (EDR) platforms, which automatically:

- Correlate related events
- Normalize telemetry
- Decode command-line arguments
- Present process execution chains in a human-readable format

---

## Continuous Monitoring Enhancements

Future enhancements to the SOC Lab include researching shell-level instrumentation techniques such as: `PROMPT_COMMAND`

```bash
export PROMPT_COMMAND='logger -p local6.notice -t bash_history "$(whoami) [$$]: $(history 1 | sed "s/^[ ]*[0-9]*[ ]*//")"'
```

to securely capture terminal activity and stream command execution data to syslog before an adversary can leverage history-evasion methods such as:

```bash
set +o history
```

Additional roadmap items include:

- Sysmon for Linux deployment
- Sigma rule development
- ATT&CK-aligned detection coverage mapping
- User behavior analytics (UBA)
- Automated alerting through Splunk correlation searches

---

## Key Takeaway

This project demonstrates how a seemingly benign insider operating with valid credentials can execute a complete attack chain—including reconnaissance, access, execution, exfiltration, and impact—while remaining largely invisible to default Linux logging configurations.

Through targeted kernel-level auditing and SIEM integration, critical process visibility gaps were eliminated, enabling meaningful detection and monitoring of insider threat activity without introducing excessive telemetry volume.

Connect with me on socials:
X (formerly twitter) : [redacted](https://x.com/thatboringbro)
