Before learning about artifact we need to know about the Query language of velociraptor. Earlier we have known that admin (security analyst) gives instruction to the client. This instructions are written in a special query language known as VQL (velociraptor query language). This is similar to SQL of MySQL. Learn more about [VQL](https://docs.velociraptor.app/docs/vql/).

Now we do not need to learn in depth about VQL. At beginner level we can just learn about artifacts and ignore VQL totally.

**Artifact:** is a packaged, reusable unit that is build around VQL. Basically artifacts are simply YAML files with VQL queries, which can be easily shared and can call other artifacts, and reuse/build on each other's logic, and can be customized by callers.
There are different types: 
- Client Artifacts run on the endpoint, 
- Client Event artifacts monitor the endpoint, 
- Server Artifacts run on the server, and 
- Server Event artifacts monitor for events on the server.
For example: let us look at the most basic artifact of velociraptor which is `Generic.Client.Info` that is used to collect basic system details including hostname, OS version, interfaces, and platform information of the client machine. Whenever new client is added to the server, this artifact automatically runs and collects information about the client. The possible YAML configuration file is:
```yaml
name: Generic.Client.Info
description: |
  Collect basic information about the client.

  This artifact is collected when any new client is enrolled into the
  system. Velociraptor will watch for this artifact and populate its
  internal indexes from this artifact as well.

  You can edit this artifact to enhance the client's interrogation
  information as required, by adding new sources.

  NOTE: Do not modify the BasicInformation source since it is used to
  interrogate the clients.

sources:
  - name: BasicInformation
    description: |
      This source is used internally to populate agent info. Do not
      modify or remove this query.
    query: |
      LET Interfaces = SELECT HardwareAddrString AS MAC
      FROM interfaces()
      WHERE HardwareAddr

      SELECT config.Version.Name AS Name,
             config.Version.BuildTime AS BuildTime,
             config.Version.Version AS Version,
             config.Version.ci_build_url AS build_url,
             config.Version.install_time AS install_time,
             config.Labels AS Labels,
             Hostname,
             OS,
             Architecture,
             Platform,
             PlatformVersion,
             KernelVersion,
             Fqdn,
             Interfaces.MAC AS MACAddresses
      FROM info()

  - name: DetailedInfo
    query: |
      LET Info = SELECT * FROM info()
      SELECT _key AS Param,
             _value AS Value
      FROM items(item=Info[0])
```
Inside this YAML file you can see query parts highlighted in green texts, which are the actual VQL, exact things that server is requesting the client to provide.
Upon successfully running it we can download result in both CSV and JSON format.
![[download dialog csv and json.png | 700]]

click on the download icon you can see in the above screenshot that you will see after clicking "Prepare CSV and JSON Download". It will download a zip folder and upon unzipping you can see a result directory. Go there and you can see all results in both CSV and JSON format like this:
![[result folder content.png | 700]]

You can see Windows Information, User Information, Basic Information.

Security analysts use a variety of artifact to inspect different aspects of client's machine, running programs, network conditions, SHA256 of multiple processes  to figure out malware's presence. Some useful artifacts are:

| Artifact                               | Description                                                                                       |
| -------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `Windows.System.DNSCache`              | Collects DNS cache entries to identify suspicious or malicious domains used for C2 communication. |
| `Windows.Analysis.EvidenceOfExecution` | Identifies evidence of process execution, useful for detecting malicious payload execution.       |
| `Windows.Analysis.EvidenceOfDownload`  | Tracks downloaded files, which could include potentially malicious payloads.                      |
| `Windows.Sysinternals.Autoruns`        | Lists autostart locations for persistence mechanisms, often used by malware.                      |
| `Windows.EventLogs.ScheduledTasks`     | Analyzes scheduled tasks to identify persistence or malicious task execution.                     |
| `Windows.Registry.RDP`                 | Detects changes to RDP-related registry keys, which could indicate remote access activity.        |
| `Windows.System.Pslist`                | Captures a list of running processes to identify suspicious ones.                                 |
| `Windows.System.UntrustedBinaries`     | Detects binaries that are not signed or have unknown origins, potentially linked to malware.      |
| `Windows.Network.NetstatEnriched`      | Provides detailed network connection data to identify unusual TCP connections.                    |
| `Windows.Attack.ParentProcess`         | Tracks parent-child process relationships to identify suspicious process spawning.                |
| `Windows.Attack.UnexpectedImagePath`   | Detects binaries executed from unexpected or non-standard locations.                              |
| `Windows.Detection.BinaryHunter`       | Searches for potentially malicious binaries present on the system.                                |
| `Windows.Detection.BinaryRename`       | Identifies renamed binaries, often a tactic used by malware for evasion.                          |
| `Windows.Detection.Impersonation`      | Detects impersonation attempts, such as stolen or misused credentials.                            |
| `Windows.Detection.Mutants`            | Identifies malicious mutex objects, often used for synchronization in malware.                    |
| `Windows.EventLogs.EvtxHunter`         | Analyzes EVTX logs for suspicious events and anomalies.                                           |
| `Windows.EventLogs.Evtx`               | Collects raw event logs for detailed manual or automated analysis.                                |
| `Windows.Memory.PEDump`                | Dumps in-memory PE files for analysis, useful for identifying injected components.                |
reference: [Daniel Jeremiah](https://daniyyell.com/threat%20hunting/tools/malware%20analysis/Using-Velociraptor-to-Detect-and-Hunt-for-Affected-Systems-Unknown-Malware-Analysis/#11-what-is-threat-hunting-)

Next Article: [[Process Hollowing & its detection using Velociraptor]]