# SOC SIEM Home-Lab

A home lab built to simulate a real-world attack and investigate it the way a SOC analyst would. The environment consists of a Windows 10 victim machine running Splunk with Sysmon for endpoint telemetry, and a Kali Linux attacker machine used to launch attacks against it.

In this scenario, a reverse TCP payload disguised as a resume is delivered to the Windows machine and executed. The resulting activity is investigated end-to-end in Splunk, from initial alert to root cause.

## Architecture

<img width="2720" height="1680" alt="soc_siem_homelab_architecture" src="https://github.com/user-attachments/assets/74cad760-03e5-44f1-8ced-f133c3351c9b" />


Sysmon on the Windows host generates detailed process, network, and file-creation telemetry. Splunk ingests these logs via the Sysmon Technology Add-on, giving the analyst a searchable record of everything that happened on the endpoint. The Kali Linux machine sits on the same NAT network and is used purely as the attacker platform — nothing is installed on it that touches the victim beyond the delivered payload and its network traffic.

## Lab Setup

1. Installed VirtualBox and downloaded Windows 10 and Kali Linux ISOs.
2. Configured both VMs on a shared NAT network so they could communicate.
3. Installed Sysmon on the Windows machine for enhanced endpoint telemetry beyond default Windows event logging.

![Sysmon installed](./Sysmon%20screenshot.jpg)

4. Installed Splunk on the Windows machine, added the Sysmon app/add-on, and configured Splunk to ingest Sysmon logs.

![Splunk installed](./Splunk%20screenshot.jpg)
![Sysmon data in Splunk](./Sysmon%20splunk.jpg)

## Attack Simulation

**Reconnaissance:** From the Kali machine, ran an Nmap scan against the Windows host to identify open ports and services.

![Nmap scan](./JPEG%20Nmap.jpg)

**Payload creation and delivery:** Generated a reverse TCP payload with msfvenom and disguised it as `Resume.pdf` to simulate a realistic social engineering delivery method.

![Reverse TCP payload](./JPEG%20reverse%20tcp.jpg)

The payload link was then opened and executed on the Windows machine, establishing a reverse shell back to the attacker. This is where the investigation begins.

## Investigation

Searched Splunk for the file name `Resume.pdf` and found 13 related events across 7 distinct event codes. Event Code 1 (process creation) had the highest number of hits, so it became the starting point for deeper investigation.

![Event code search](./Event%20code.jpg)

Drilling into the Event Code 1 results showed that the payload's parent process had spawned `cmd.exe` with PID 8308 — a strong indicator of shell access gained through the malicious file.

![Parent process ID](./ParentImage%20ID%20.jpg)
![Spawned process](./Spawned%20process.jpg)

Pivoting on the process GUID revealed the full sequence of commands run through that shell: `net user`, `net localgroup`, and `ipconfig` — classic post-exploitation discovery commands used to enumerate accounts, group membership, and network configuration.

![Investigation findings](./Investigation%20findings.jpg)

## MITRE ATT&CK Mapping

| Attack Step | Technique | ATT&CK ID | Tactic |
|---|---|---|---|
| Nmap scan against Windows host | Network Service Discovery | T1046 | Discovery |
| msfvenom payload disguised as "Resume.pdf" | Masquerading | T1036 | Defense Evasion |
| Payload delivery & execution on victim | User Execution: Malicious File | T1204.002 | Execution |
| Reverse TCP shell established | Application Layer Protocol | T1071.001 | Command and Control |
| cmd.exe spawned from payload process | Command and Scripting Interpreter: Windows Command Shell | T1059.003 | Execution |
| `net user` executed | Account Discovery: Local Account | T1087.001 | Discovery |
| `net localgroup` executed | Permission Groups Discovery: Local Groups | T1069.001 | Discovery |
| `ipconfig` executed | System Network Configuration Discovery | T1016 | Discovery |

## Key Findings

- The malicious file successfully evaded casual inspection by masquerading as a harmless PDF resume.
- Execution of the payload spawned a command shell, giving the attacker interactive access to the host.
- Post-exploitation activity focused on discovery (accounts, groups, network config) — consistent with an attacker performing initial reconnaissance after gaining a foothold.
- Sysmon's process, parent-process, and command-line logging was essential to reconstructing the full attack chain; default Windows event logging alone would not have provided this level of detail.


## Tools Used

- VirtualBox
- Kali Linux
- Windows 10
- Sysmon
- Splunk
- Metasploit / msfvenom
- Nmap

## Incident Report

[#incident-report](#incident-report)

A formal incident report was written up for this exercise, following the same structure and rigor used for a real SOC incident (executive summary, timeline, detection & analysis, IOCs, MITRE ATT&CK mapping, impact assessment, containment/eradication/recovery, and recommendations).

| Field | Value |
|---|---|
| **Incident ID** | IR-2026-001 |
| **Analyst** | Maurice Ferguson |
| **Severity** | High (simulated) |
| **Status** | Closed — Contained & Documented |
| **Category** | Malicious Code Execution / Post-Exploitation Discovery |

### Executive Summary

A Windows 10 host in this lab executed a malicious file disguised as `Resume.pdf`, resulting in a reverse shell connection back to an attacker-controlled machine. The activity was identified and investigated using Splunk (SIEM) with Sysmon-enhanced endpoint telemetry. Analysis confirmed the executable spawned a command shell and was used to run several discovery commands consistent with post-exploitation reconnaissance. No lateral movement, data exfiltration, or persistence mechanisms were observed. The incident was fully reconstructed from initial delivery through post-exploitation activity using process ancestry and command-line analysis, and mapped to eight MITRE ATT&CK techniques across five tactics.

### Detection & Analysis

**Initial Search:** The investigation began with a Splunk search for the delivered filename, `Resume.pdf`. This search returned 13 related events spanning 7 distinct Sysmon event codes. Event Code 1 (Process Creation) had the highest number of hits and was selected as the starting point for deeper investigation.

**Process Ancestry:** Drilling into the Event Code 1 results showed that the payload's parent process had spawned `cmd.exe` (PID 8308) — a strong indicator that the malicious file had granted the attacker interactive shell access to the host.

**Command-Line Reconstruction:** Pivoting on the process GUID associated with the spawned shell revealed the full sequence of commands executed by the attacker: `net user`, `net localgroup`, and `ipconfig` — discovery commands used to enumerate local accounts, group membership, and network configuration.

**Analyst Assessment:** No evidence of lateral movement, credential dumping, additional payload staging, or persistence mechanisms was identified in the available telemetry. The observed activity is consistent with the early discovery phase of an intrusion.

### Indicators of Compromise (IOCs)

- **Filename:** `Resume.pdf` (masquerading as a benign document; actual payload was a reverse TCP executable generated with msfvenom)
- **Spawned process:** `cmd.exe`, PID 8308, launched as a child of the executed payload
- **Post-exploitation commands:** `net user`, `net localgroup`, `ipconfig`
- **Behavioral pattern:** Reverse shell (Application Layer Protocol C2) followed immediately by local discovery commands

### MITRE ATT&CK Mapping

| Attack Step | Technique | ATT&CK ID | Tactic |
|---|---|---|---|
| Nmap scan against Windows host | Network Service Discovery | T1046 | Discovery |
| Payload disguised as "Resume.pdf" | Masquerading | T1036 | Defense Evasion |
| Payload delivery & execution | User Execution: Malicious File | T1204.002 | Execution |
| Reverse TCP shell established | Application Layer Protocol | T1071.001 | Command and Control |
| cmd.exe spawned from payload | Command & Scripting Interpreter: Windows Shell | T1059.003 | Execution |
| `net user` executed | Account Discovery: Local Account | T1087.001 | Discovery |
| `net localgroup` executed | Permission Groups Discovery: Local Groups | T1069.001 | Discovery |
| `ipconfig` executed | System Network Configuration Discovery | T1016 | Discovery |

### Impact Assessment

Within the scope of this lab, the attacker achieved interactive shell access to a single host and completed local discovery reconnaissance. Had this occurred in a production environment, the enumerated account and group information could have been used to plan credential-based lateral movement or privilege escalation. Because default Windows Event Logging alone does not capture process ancestry or command-line arguments in sufficient detail, an environment without Sysmon-level telemetry would likely have missed the parent-child process relationship and command reconstruction that made root-cause analysis possible here.

### Containment, Eradication & Recovery

- **Containment:** The lab network is isolated by design (NAT-only VirtualBox network), so no additional network containment was required; in production, the recommended action would be immediate host isolation from the network.
- **Eradication:** The malicious executable and spawned process were identified and would be removed/terminated; the host would be flagged for a full antivirus/EDR scan and re-imaging if persistence were suspected.
- **Recovery:** In a live environment, the affected host would be monitored closely post-remediation for reappearance of the same process tree, filename, or C2 behavior before being returned to normal use.

### Recommendations & Lessons Learned

- **Deploy Sysmon (or equivalent EDR telemetry) organization-wide.** Default Windows Event Logging did not provide the process ancestry or command-line visibility needed to reconstruct this attack chain; Sysmon was the deciding factor in identifying root cause.
- **Alert on masquerading file types.** Consider detections for executables with document-style filenames/extensions (e.g., ".pdf.exe" or icon spoofing) delivered via email or removable media.
- **Tune detections around discovery commands.** `net user`, `net localgroup`, and `ipconfig` are low-noise individually, but a short burst of all three from a freshly spawned shell is a high-fidelity indicator worth alerting on.
- **User awareness training.** This scenario relied on social engineering (a disguised "resume" file); reinforcing user caution around unsolicited attachments remains a low-cost, high-value control.
- **Baseline this attack chain as a detection use case.** The full kill chain documented here (recon → masquerading → execution → C2 → discovery) is a strong candidate for a saved Splunk correlation search or detection rule.

### Conclusion

This investigation demonstrated a complete SOC analyst workflow: identifying a suspicious file execution, pivoting through endpoint telemetry to establish process ancestry, reconstructing attacker command-line activity, and mapping observed behavior to the MITRE ATT&CK framework. The exercise reinforced the value of high-fidelity endpoint telemetry (Sysmon) in reducing investigation time and improving accuracy of root-cause analysis.
