According to [IBM](https://www.ibm.com/think/topics/threat-hunting) Threat Hunting is a proactive (prevention before occurrence happens) approach, where security analysts search through networks, endpoints and datasets to detect hidden malicious activity that has bypasses automated security tools.
Unlike traditional reactive (opposite to proactive; where to find cure after the occurrence happened) approaches, threat hunting assumes an attacker is already inside the environment and uses human expertise to uncover stealthy threats.

### Threat Hunting Catagory
Based on the initiation of threat hunting it is categorized in two segment: 
1. **Reactive Threat Hunting:** 
	a post-incident security strategy triggered by external intelligence or suspicious alerts. Instead of looking for hidden, preemptive threats analysts take active in response to specific indicators, alerts and intelligence. Focuses on investigating known threats or alerts. 

2. **Proactive Threat Hunting:** 
	is the analyst-driven practice of actively searching through networks and systems to detect stealthy cyber threats that bypass automated defenses. By assuming a breach has already occurred, hunters minimize dwell time and stop adversaries before they cause business disruption.

Based on methodology (how the hunt is conducted) the threat hunting is divided in three ways:
1. **Hypothesis Driven Threat Hunting:** 
	is a proactive cyber-security methodology where security analysts formulate an educated guess about potential adversary behavior and query their environment to prove or disprove it. Instead of waiting for automated alerts, this method targets attacker Tactics, Techniques, and Procedures (TTPs) using frameworks like.
	This technique consists of four step cycle:
		**Formulate the Hypothesis:** Based on threat intelligence, recent red team findings , or environmental vulnerabilities , create a testable assumption. For example: "Adversaries are using scheduled tasks for lateral movement."
		**Search and Investigate:** Query your data sources (e.g., endpoint telemetry, network traffic logs, cloud events) to look for behavioral indicators that confirm the assumption.
		**Analyze Findings:** Determine if the discovered anomalies or events are benign, misconfigurations, or true positive threats.
		**Respond and Refine:** If a threat is found, initiate incident response . If the hypothesis is refuted , use the gathered telemetry to improve automated detection rules.

2. **IOC based Threat hunting** 
	is one kind of reactive hunting. IOC is Indicators for Compromise. According to Microsoft: An **indicator of compromise (IOC)** is evidence that someone may have breached an organization’s network or endpoints. This forensic data doesn’t just indicate a potential threat, it signals that an attack, such as malware, compromised credentials, or data **exfiltration** (is the unauthorized, stealthy transfer or theft of data from a compromised computer, device, or network). 

	It represents the final, critical stage of a cyber-attack where criminals successfully remove sensitive information—such as intellectual property, financial records, or user credentials—to an external location.), has already occurred. 

	Security professionals search for IOCs on event logs, Extended Detection and Response solutions, and Security Information and Event Management solutions. During an attack, the team uses IOCs to eliminate the threat and mitigate damage. 

	After recovery, IOCs help an organization better understand what happened, so the organization’s security team can strengthen security and reduce the risk of another similar incident.

	IOC can be:
	- Network traffic anomalies
	- Unusual Sign in attempts
	- Privilege account irregularities
	- Changes to system configurations
	- Unexpected software installations or updates
	- Numerous requests for the same files
	- Unusual DNS requests

3. **Event Driven (Behavioral or Situational) Threat Hunting:**
	it is a proactive cybersecurity approach triggered by specific environmental events or intelligence indicators, rather than a continuous or scheduled hypothesis. It zeroes in on highly risky entities to quickly contain emerging, targeted threats.

	Instead of looking for known malware or IOCs, the hunter looks for **abnormal behavior**. This means the most valuable targets are always being monitored and the risk of big breaches is reduced.

4. **Unknown Threat Hunting:** 
	Hunt begins without specific threat in mind. Aims to uncover unknown threats. Its more like surfing the web and coincidently find something useful.

We say, "prevention is better than cure" and such proactive threat hunting is better approach to safeguard a project. Security analysts hunts for signs of infiltration, assuming actors already have breached the perimeter or gained access through a vulnerability.

### Threat Hunting Tools Category

|Category|Purpose|Examples|
|---|---|---|
|Endpoint Detection & Response (EDR)|Collect endpoint telemetry, detect threats, and investigate hosts|Velociraptor, Microsoft Defender for Endpoint, CrowdStrike Falcon, SentinelOne|
|Security Information and Event Management (SIEM)|Aggregate and analyze logs from many sources|Splunk, Microsoft Sentinel, QRadar, Elastic SIEM|
|Network Detection & Response (NDR)|Analyze network traffic and detect suspicious communications|Zeek, Suricata, Corelight, Darktrace|
|Digital Forensics & Incident Response (DFIR)|Collect and analyze forensic evidence|Velociraptor, Autopsy, FTK, EnCase|
|Threat Intelligence Platforms (TIP)|Manage IOCs and threat intelligence|MISP, OpenCTI, ThreatConnect|
|Log Management|Store and search logs|Elasticsearch, Graylog, Loki|
|SOAR|Automate investigations and responses|Cortex XSOAR, Splunk SOAR, Tines|
|Malware Analysis|Reverse engineer or analyze malware|Ghidra, IDA Pro, REMnux, Cuckoo Sandbox|
|Memory Forensics|Analyze RAM for malicious activity|Volatility, Rekall|

### Velociraptor
Velociraptor is primarily an Endpoint DFIR (Digital Forensics & Incident Response) and Threat Hunting platform. It overlaps with several categories such as:
it is a Endpoint Threat hunting tool because it queries endpoints for processes, files, registry, users, services, etc. Then it is a DFIR tool because it's main purpose is forensic evidence collection and incident investigation. Lastly it is partially a EDR tool because it provides endpoint visibility and live response, though it's generally considered a DFIR-focused EDR rather than a commercial prevention-focused EDR. 

**What velociraptor does?** 
Velociraptor installs a lightweight **client** on endpoints that communicates with a **server**.
The server can ask clients questions such as:

`Are any users running PowerShell?`
`Find every executable created in the last 24 hours.`
`Collect browser history from every workstation.`
`Show all scheduled tasks.`
`Collect Windows Event Logs.`

Instead of waiting for logs to arrive, Velociraptor **actively queries endpoints** and returns the results.

**Why it is excellent for threat hunting?**
Imagine you suspect attackers are using encoded PowerShell commands.
With Velociraptor, you can run a hunt across **thousands of endpoints** asking:
Find every process where:

`Process Name = powershell.exe`
`AND`
`Command Line contains "-EncodedCommand"`

Every client executes the query locally and sends the results back to the server.
This is a classic example of **endpoint threat hunting**.

**Velociraptor's Main Capabilities**
- Live endpoint interrogation
- Remote forensic collection
- Process and service inspection
- File system searches
- Windows Registry analysis
- Event log collection
- Memory acquisition (with appropriate artifacts/tools)
- Artifact-based hunting
- Remote execution of VQL (Velociraptor Query Language)

**Why is it a DIFR tool?**
Velociraptor was originally designed to support **Digital Forensics and Incident Response (DFIR)**.
For example, if ransomware infects a workstation, an investigator can use Velociraptor to:
1. Collect Windows event logs.
2. Retrieve the Master File Table (MFT).
3. Examine running processes.
4. Check persistence mechanisms (services, scheduled tasks, registry autoruns).
5. Collect browser history.
6. Acquire memory (if configured).
7. Search for indicators of compromise across many hosts.
These capabilities make it invaluable for post-compromise investigations.

>[!tip]
>You may find similarities with autopsy and velociraptor. But keep in mind that autopsy is hard drive oriented and cannot be used in multiple devices in one session. Each end user needs individual autopsy session. On the other hand velociraptor is a centralized system. We have one server and thousands on endpoints or users. 
<div align="center">  
<img src="velociraptor server &amp; client.png" width="700">  
</div>


Next Article: [[Velociraptor Setup & Artifact Collection]]