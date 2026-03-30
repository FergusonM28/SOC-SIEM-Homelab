# SOC-SIEM-Homelab
 I have created a home lab in a sandbox environment that includes a Windows machine with a Splunk SIEM configured & Sysmon for added telemetry. I have also configred a separate Kali Linux machine to test different attacks against the Windows machine.

 
 
 
 I downloaded VirtualBox along with Kali Linux & Windows ISO files. Also setup NAT connection for both
 
 Installed Sysmon for additional telemetry 
![image alt](https://github.com/FergusonM28/SOC-SIEM-Homelab/blob/dfdc2738c1bdae21c33435b1551969790c75a28d/Sysmon%20screenshot.jpg)
 
Downloaded Splunk & added the Sysmon app addition first then configured Splunk to ingest Sysmon logs
![image alt](https://github.com/FergusonM28/SOC-SIEM-Homelab/blob/5b16de108a968ccdf544a59398dcc8d708fc8ca3/Splunk%20screenshot.jpg)
![image alt](https://github.com/FergusonM28/SOC-SIEM-Homelab/blob/a3e65a899d98c8efd149a12789819e13e39ef082/Sysmon%20splunk.jpg)

On the Kali Linux attacker machine, I ran an NMAP scan on the Windows machine to check for open ports ![image alt](https://github.com/FergusonM28/SOC-SIEM-Homelab/blob/f707fc6bb870a5716138745b3a89788904fc07dd/JPEG%20Nmap.jpg)

I then made a reverse tcp payload using msfvenom against the Windows machine & named it "Resume.pdf" ![image alt](https://github.com/FergusonM28/SOC-SIEM-Homelab/blob/4eb325c778f78594b713044b16fea539c16eeb9e/JPEG%20reverse%20tcp.jpg)

Once I generated a link with the malware I created I went over to the Windows machine & executed it, this is where the investigation begins!

I queried the file name, "Resume.pdf", & 13 related events showed up. Of those 13 events, 7 event codes came up with event code number 1 having the most hits so I further investigated that ![image alt](https://github.com/FergusonM28/SOC-SIEM-Homelab/blob/e1a03107a98909c4ecbe4fb7df858c7bbcfab4a5/Event%20code.jpg)

During the search I found the ParentImage had spawned the process of cmd.exe with a PID of 8308 ![image alt](https://github.com/FergusonM28/SOC-SIEM-Homelab/blob/9ff44957c313bdb56d1d1a45508125dca29d6320/ParentImage%20ID%20.jpg)
![image alt]()
