# SOC SIEM Homelab

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

## Skills Demonstrated

- Building and configuring a Windows/Linux lab environment in VirtualBox
- Installing and tuning Sysmon for endpoint telemetry
- Ingesting and searching logs in Splunk
- Simulating an attack chain (recon → payload delivery → execution → post-exploitation)
- Investigating an alert from initial indicator through to root cause using process ancestry and command-line analysis
- Mapping observed attacker behavior to the MITRE ATT&CK framework

## Tools Used

- VirtualBox
- Kali Linux
- Windows 10
- Sysmon
- Splunk
- Metasploit / msfvenom
- Nmap
