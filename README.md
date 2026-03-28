# Wazuh SIEM Home Lab

A two-machine Security Information and Event Management (SIEM) lab built to develop hands-on skills in log analysis, threat detection, and incident triage using industry-standard tooling.

---

## Architecture

| Machine | Role | OS |
|---|---|---|
| Lenovo ThinkPad P43s | Wazuh Server (Manager + Indexer + Dashboard) | Linux Mint |
| Custom Gaming PC (RTX 3080) | Monitored Endpoint (Wazuh Agent) | Windows 11 |

Both machines on the same local network, with the ThinkPad assigned a static IP via DHCP reservation on the AT&T gateway.

```
[Windows Agent] --1514/UDP--> [Wazuh Manager]
[Windows Agent] --1515/TCP--> [Wazuh Manager]
[Browser]       <--443/HTTPS- [Wazuh Dashboard]
```

---

## Components Deployed

- **Wazuh Manager 4.14** — Receives and processes agent events, applies detection rules, generates alerts
- **Wazuh Indexer** — OpenSearch-based data store for all alerts and events
- **Wazuh Dashboard** — Web UI for alert triage, threat hunting, and MITRE ATT&CK mapping
- **Wazuh Agent (Windows)** — Installed on Windows endpoint, ships logs and events to the manager

---

## What I Built

### Server Setup (Linux Mint)
- Deployed Wazuh all-in-one stack using the official installation assistant
- Configured `config.yml` with static IP addressing for all three nodes
- Generated SSL certificates for encrypted agent-server communication
- Configured UFW firewall rules to allow agent traffic (1514, 1515, 1516, 55000)
- Resolved dpkg package conflicts during installation via forced removal and reinstallation

### Windows Agent Setup
- Deployed Wazuh Agent on Windows endpoint via PowerShell MSI install
- Registered agent to the Wazuh Manager using auto-enrollment on port 1515
- Enabled verbose Windows audit policies for enhanced log coverage:
  - Logon/Logoff events (Event ID 4624, 4625)
  - Process creation (Event ID 4688)
  - Privilege use

### Network Configuration
- Assigned static IP to Wazuh server via router DHCP reservation
- Opened required firewall ports on both server (UFW) and agent (Windows Firewall)
- Validated bidirectional connectivity between machines prior to agent enrollment

---

## Detection Testing

### Authentication Failure Detection
**Objective:** Validate that Wazuh correctly detects and alerts on failed Windows login attempts.

**Method:** Triggered multiple failed login attempts (Event ID 4625) on the Windows endpoint.

**Result:** Wazuh generated authentication failure alerts, visible in Threat Hunting dashboard. Events mapped to MITRE ATT&CK **T1110 - Brute Force** under the Valid Accounts tactic.

**Observation:** Windows batches Event ID 4625 entries and flushes them to the log upon successful authentication (4624) — a pattern consistent with successful credential stuffing attacks in real environments.

---

## Dashboard Findings

After initial deployment and testing, the Wazuh dashboard showed:

- **11,182 total events** collected across both agents
- **4 authentication failures** detected
- **34 authentication successes** logged
- **MITRE ATT&CK techniques identified:** Valid Accounts, Account Manipulation, Windows Service, Domain Policy Modification
- **Both agents active:** `boxedmilktea` (Windows), `eyebot` (ThinkPad/Linux)

 ## Dashboard Screenshot
![Wazuh Dashboard](Screenshot_2026-03-28_at_04-03-32_Wazuh.png)

---

## Skills Demonstrated

- SIEM deployment and configuration
- Linux system administration (systemd, UFW, dpkg, apt)
- Windows security event log analysis
- Network configuration (static IP, DHCP reservation, port management)
- Firewall rule management on both Linux and Windows
- Agent enrollment and registration
- MITRE ATT&CK framework familiarity
- Troubleshooting complex multi-component installation failures

---

## Tools & Technologies

`Wazuh 4.14` `OpenSearch` `Linux Mint` `Windows 11` `UFW` `systemd` `PowerShell` `MITRE ATT&CK`

---

## Troubleshooting Encountered

| Issue | Root Cause | Resolution |
|---|---|---|
| `wazuh-install.sh` not found | Incorrect version path (4.9 vs 4.14) | Updated URL to packages.wazuh.com/4.14 |
| Invalid IP in config.yml | IP wrapped in angle brackets `<ip>` | Removed brackets, bare IP only |
| Wazuh Manager not found after install | `wazuh-agent` package conflicted with manager install | Force-removed agent via `dpkg --remove --force-all`, cleared dpkg info, reinstalled manager |
| Agent queue flood alert | Agent backlog on first connection | Self-resolved after initial sync |
| Dashboard AxiosError 3000 | Manager service not running | Reinstalled wazuh-server component with `--overwrite` flag |

---

## Next Steps

-  Install Sysmon on Windows endpoint with SwiftOnSecurity config for richer telemetry
-  Configure File Integrity Monitoring (FIM) on sensitive Windows directories
-  Write custom detection rules in `/var/ossec/etc/rules/`
-  Simulate attack scenarios using Metasploit and document triage process
-  Enable vulnerability detection module and analyze CVE findings
