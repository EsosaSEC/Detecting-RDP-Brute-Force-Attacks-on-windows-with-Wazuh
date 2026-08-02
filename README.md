# Detecting and Investigating an RDP Brute-Force Attack Using Wazuh

## Overview

This project demonstrates the complete detection lifecycle of an RDP brute-force attack in a controlled lab environment.

A brute-force attack was simulated from a Kali Linux attacker machine using **Hydra** against a **Windows 10** endpoint. The attack was successfully detected by **Wazuh**, investigated using **Windows Security Event Logs**, and mapped to the **MITRE ATT&CK Framework**.

This project was built to develop practical Security Operations Center (SOC) skills, including attack simulation, log analysis, threat detection, and incident investigation.

---

## Objectives

- Configure Wazuh to collect Windows Security and Sysmon logs
- Simulate failed authentication attempts
- Execute an RDP brute-force attack using Hydra
- Detect authentication failures in Wazuh
- Investigate Windows Security Events
- Map detections to the MITRE ATT&CK Framework
- Recommend mitigations against brute-force attacks

---

# Lab Environment

| Component | Technology |
|-----------|------------|
| SIEM | Wazuh |
| Attacker Machine | Kali Linux |
| Target Machine | Windows 10 Pro |
| User Account | esosa |
| Attack Tool | Hydra |
| Remote Access | RDP (TCP 3389) |

---

# Lab Architecture

```
                +----------------------+
                |     Kali Linux       |
                |       Hydra          |
                +----------+-----------+
                           |
                   RDP Brute Force
                           |
                           ▼
                +----------------------+
                |     Windows 10       |
                |     RDP Service      |
                +----------+-----------+
                           |
                 Windows Security Logs
                           |
                           ▼
                +----------------------+
                |    Wazuh Agent        |
                +----------+-----------+
                           |
                           ▼
                +----------------------+
                |   Wazuh Manager       |
                +----------+-----------+
                           |
                           ▼
                +----------------------+
                |  Wazuh Dashboard      |
                +----------------------+
```

---

# Environment Preparation

## Windows 10

- Installed the Wazuh Agent
- Installed Sysmon
- Enabled Remote Desktop
- Verified RDP was listening on TCP Port **3389**
<img width="619" height="518" alt="Screenshot 2026-05-27 121354" src="https://github.com/user-attachments/assets/62e36e21-8a27-4f21-809e-8396a93c956b" />
<img width="559" height="445" alt="Screenshot 2026-05-27 160018" src="https://github.com/user-attachments/assets/9a7a22c3-7a0a-45bd-bcf6-4a70790fe8c6" />
<img width="800" height="425" alt="Screenshot 2026-05-27 160715" src="https://github.com/user-attachments/assets/f67b34f7-76e6-4045-b7dc-63993a7a5e9a" />
<img width="1165" height="392" alt="Screenshot 2026-05-27 162113" src="https://github.com/user-attachments/assets/9fbfbbdf-35c4-4409-9736-9bbb4350f011" />

```powershell
netstat -ano | findstr :3389
```
<img width="373" height="155" alt="Screenshot 2026-07-28 125722" src="https://github.com/user-attachments/assets/f88e76a6-1d1b-465c-aa18-c9ea21d092e6" />
---

### Windows Firewall

Confirmed that the Windows Defender Firewall allowed inbound Remote Desktop connections.

<img width="646" height="52" alt="Screenshot 2026-07-30 202728" src="https://github.com/user-attachments/assets/0282d063-646b-4eaa-ac18-17d3d2df8791" />


---

## Wazuh Configuration

Configured Wazuh to collect:

- Windows Security Event Logs
- Sysmon Operational Logs

Verified successful log ingestion by generating Windows events and confirming they appeared in the Wazuh Dashboard.

---

# Attack Simulation

## Phase 1 – Manual Failed Authentication

Before launching the attack, multiple failed logins were intentionally generated on Windows.

This confirmed that:

- Windows generated **Event ID 4625**
- Wazuh successfully ingested the authentication events

---

## Phase 2 – RDP Brute Force Using Hydra

Hydra was executed from Kali Linux against the Windows RDP service.

```bash
hydra -l esosa -P passwords.txt rdp://192.168.50.11 -t 2
```

### Hydra Output

<img width="901" height="263" alt="Screenshot 2026-07-28 140611" src="https://github.com/user-attachments/assets/1f20d229-6a6e-429b-8153-1fc84a131fed" />
<img width="214" height="222" alt="Screenshot 2026-07-28 140634" src="https://github.com/user-attachments/assets/0999fc92-a91f-4dbb-871f-9c7e22904672" />
<img width="214" height="222" alt="Screenshot 2026-07-28 140634" src="https://github.com/user-attachments/assets/5460e962-a6f5-4f60-b4e8-e4e2264f22b5" />


The attack generated multiple failed authentication attempts within a short period.

---

# Detection

## Windows Security Logs

The primary Windows event generated was:

| Event ID | Description |
|----------|-------------|
| 4625 | An account failed to log on |

### Key Event Fields

| Field | Description |
|--------|-------------|
| Account Name | esosa |
| Source IP Address | Kali Linux |
| Logon Type | 3 |
| Failure Reason | Invalid Credentials |
| Timestamp | Attack Time |

**Screenshot**
<img width="214" height="222" alt="Screenshot 2026-07-28 140634" src="https://github.com/user-attachments/assets/11a6d8c7-3f8a-49aa-b54c-a2a7077fa08d" />

---

## Wazuh Detection

Wazuh successfully detected the repeated authentication failures.

The alerts were mapped to the MITRE ATT&CK Framework.

| Tactic | Technique |
|---------|-----------|
| Credential Access | T1110 – Brute Force |

**Screenshot**

<img width="959" height="390" alt="Screenshot 2026-07-28 150226" src="https://github.com/user-attachments/assets/aafa18df-7976-42a9-bafc-fa494a17be35" />
<img width="650" height="343" alt="Screenshot 2026-07-28 145955" src="https://github.com/user-attachments/assets/a6451f78-e48c-491b-880d-9badec8afc00" />
<img width="941" height="351" alt="Screenshot 2026-07-28 150125" src="https://github.com/user-attachments/assets/ff6f6f6b-f002-4726-9a42-aa30fbe2e7dd" />
<img width="955" height="102" alt="Screenshot 2026-07-28 150202" src="https://github.com/user-attachments/assets/bae17fba-992b-431d-b39a-8081efbc7029" />

---

# Investigation

The following information was collected during the investigation.

| Field | Value |
|--------|-------|
| Source IP | Kali Linux |
| Target Host | Windows 10 |
| Target Username | esosa |
| Windows Event ID | 4625 |
| Logon Type | 3 |
| Detection | Failed Logon |
| MITRE Technique | T1110 – Brute Force |

Evidence collected included:

- Hydra output
- Windows Event Viewer
- Wazuh Alerts
- MITRE ATT&CK Mapping

---

# MITRE ATT&CK Mapping

| Tactic | Technique | Description |
|---------|-----------|-------------|
| Credential Access | T1110 – Brute Force | Multiple failed RDP authentication attempts |

---

# Timeline

| Time | Activity |
|------|----------|
| T0 | Lab environment configured |
| T1 | Manual failed authentication attempts |
| T2 | Hydra brute-force attack |
| T3 | Wazuh detected authentication failures |
| T4 | Investigation and MITRE mapping |

---

# Recommendations

- Enable Multi-Factor Authentication (MFA)
- Implement strong password policies
- Configure account lockout thresholds
- Restrict RDP access using VPN or firewall rules
- Monitor Windows Event ID **4625**
- Configure correlation rules for brute-force detection
- Regularly review Wazuh alerts and MITRE mappings

---

# Lessons Learned

Throughout this lab I learned how to:

- Configure Wazuh to collect Windows Security and Sysmon logs
- Detect RDP brute-force attacks
- Investigate Windows Security Event ID 4625
- Analyze alerts using the Wazuh Dashboard
- Map detections to the MITRE ATT&CK Framework
- Perform practical SOC investigation workflows

---

# Skills Demonstrated

- Security Monitoring
- Threat Detection
- Windows Event Analysis
- Wazuh SIEM
- Sysmon
- Hydra
- MITRE ATT&CK
- Incident Investigation
- Blue Team Operations

---

# Future Improvements

- Create custom Wazuh detection rules for brute-force attacks
- Configure Active Response to automatically block malicious IP addresses
- Build a dedicated Wazuh dashboard for authentication attacks
- Integrate email or Slack alerting
- Extend the lab to Active Directory environments

---

# References

- Wazuh
- Sysmon
- Microsoft Windows Security Auditing
- MITRE ATT&CK Framework

---

## Author

**Esosa Okonedo**

Cybersecurity | SOC Analyst
