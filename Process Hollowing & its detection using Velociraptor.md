**Process Hollowing:** According to [MITRE ATT&CK](https://attack.mitre.org/techniques/T1055/012/) Process Hollowing is technique where the malware creates a legitimate process in a suspended state, that removes or "hollows out" its legitimate code from memory and replaces its malicious code before resuming execution. This lets malicious code run under the guise of a trusted process name such as `explorer.exe` or `svchost.exe`. 
This peculiar technique helps the malware evade detection by security tools that check process names or signatures rather than actual memory content.

**Why it is used?**
- Evades antivirus or EDR tools that whitelist trusted system process.
- Hides malicious activity in task managers, since the process appears legitimate.
- Bypasses some application whitelisting controls.

**How it works?**
- Create a legitimate process in suspended state
- Overwrite its memory image
- Write malicious code into the memory space
- Resume the thread so the malicious code executes.

