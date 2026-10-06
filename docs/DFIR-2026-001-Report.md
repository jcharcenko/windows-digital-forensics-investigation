# Windows Digital Forensics Investigation

**Case ID:** DFIR-2026-001  
**Evidence System:** WIN11-DFIR  
**Examiner:** Jevgenij Charcenko  
**Investigation Type:** Controlled Windows forensic investigation  
**System Timezone:** Europe/Dublin  
**Investigation Date:** 5 October 2026  

## 1. Executive Summary

This investigation examined a Windows 11 workstation following simulated suspicious PowerShell activity in a controlled DFIR laboratory environment.

The objective was to reconstruct the sequence of activity using forensic artifacts acquired from the endpoint and volatile memory rather than relying solely on the original security monitoring data.

The investigation identified evidence of a PowerShell script being transferred to the workstation, executed by the user, used in the creation of a user-level persistence mechanism, and subsequently opened in Notepad. A temporary file was also created and deleted during the incident window.

Evidence was correlated across multiple forensic sources including Windows event logs, Sysmon, Prefetch, Registry artifacts, NTFS metadata, the USN Journal, PowerShell command history, shortcut artifacts, and volatile memory.

Six principal findings were established with high confidence.

## 2. Scope and Objectives

The investigation sought to determine:

1. What activity occurred on the workstation?
2. Which user and processes were associated with the activity?
3. What files were created, executed, or deleted?
4. Whether persistence or other system changes occurred.
5. Whether network-related activity could be reconstructed.
6. What evidence remained available in volatile memory.
7. Whether findings could be independently corroborated across multiple forensic artifacts.

The investigation was performed as a controlled benign simulation. No malware was deployed.

## 3. Forensic Methodology

The investigation followed a structured evidence-driven workflow:

1. Preserve the post-incident system state.
2. Acquire volatile memory.
3. Collect relevant endpoint artifacts.
4. Record cryptographic hashes for acquired evidence where applicable.
5. Parse individual forensic artifacts using appropriate forensic tools.
6. Normalize and correlate timestamps.
7. Construct a consolidated forensic timeline.
8. Cross-validate significant findings using independent evidence sources.
9. Record negative findings and tool limitations.
10. Separate observed evidence from investigative interpretation.

Evidence and analysis outputs were maintained separately within the case workspace.

## 4. Evidence Sources

The investigation used the following evidence sources:

- Volatile memory
- Windows Sysmon event logs
- Windows Security and PowerShell artifacts
- Windows Prefetch
- User Registry hive
- NTFS Master File Table
- NTFS USN Journal
- NTFS transaction log artifacts
- PowerShell ConsoleHost history
- Windows shortcut artifacts
- KAPE logical collection
- Autopsy artifact analysis

## 5. Findings

| ID | Finding | Assessment | Confidence | Primary Evidence |
|---|---|---|---|---|
| F-001 | PowerShell Script Acquisition | Confirmed | High | PowerShell history, USN Journal, SHA-256 |
| F-002 | PowerShell Script Execution | Confirmed | High | Sysmon, Prefetch, USN Journal, PowerShell history |
| F-003 | User-Level Registry Persistence | Confirmed | High | Sysmon, NTUSER.DAT, RECmd, Prefetch |
| F-004 | Temporary File Creation and Deletion | Confirmed | High | USN Journal, NTFS $LogFile, PowerShell history |
| F-005 | Script Opened in Notepad | Confirmed | High | Sysmon, Prefetch, Volatility, LNK artifact |
| F-006 | Controlled Network Transfer | Confirmed | High | PowerShell history, USN Journal, SHA-256 correlation |

### F-001 - PowerShell Script Acquisition

The file update-check.ps1 was transferred from the controlled test server at 192.168.134.128 to the Downloads directory of the dfiruser account. PowerShell history recorded the request, the USN Journal recorded creation of the downloaded file, and SHA-256 comparison confirmed that the downloaded artifact matched the source artifact.

### F-002 - PowerShell Script Execution

The downloaded script was executed using powershell.exe with ExecutionPolicy Bypass. Sysmon captured the complete command line, Prefetch corroborated PowerShell execution, and USN records showed creation of update.log and execution-time.txt immediately afterwards.

### F-003 - User-Level Registry Persistence

A Registry Run-key value named UpdateCheck was created under the dfiruser account. Sysmon recorded the reg.exe command responsible for creating the value, while independent examination of NTUSER.DAT confirmed that the persistence entry existed in the acquired Registry hive.

### F-004 - Temporary File Creation and Deletion

project-notes.txt was created, written to, and deleted from the user's Documents directory. Its lifecycle was preserved in the USN Journal despite the corresponding filename no longer being available through the parsed MFT. NTFS $LogFile and PowerShell history provided additional corroboration.

### F-005 - Script Opened in Notepad

update-check.ps1 was opened with Notepad. This activity was independently supported by Sysmon, Prefetch, PowerShell history, shortcut evidence, and the Notepad command line recovered from volatile memory.

### F-006 - Controlled Network Transfer

Endpoint artifacts support successful retrieval of update-check.ps1 from 192.168.134.128:8080. No corresponding live connection was recovered by Volatility netscan approximately 22 minutes later. The negative memory result was therefore treated as a limitation of the acquisition-time network state rather than evidence that the earlier transfer did not occur.

## 6. Forensic Timeline

Timestamps from the principal forensic artifacts were normalized to UTC for correlation. The investigated Windows system was configured for Europe/Dublin, which was UTC+1 (IST) during the incident.

| UTC Time | Evidence Source | Reconstructed Activity |
|---|---|---|
| 20:41:12.985 | USN Journal | update-check.ps1 created in the dfiruser Downloads directory |
| 20:42:30.117 | Sysmon / Prefetch / USN | update-check.ps1 executed through PowerShell; script-generated filesystem activity followed |
| 20:43:05.193 | Sysmon / Prefetch / Registry | UpdateCheck Run-key persistence established |
| 20:43:41.628 | USN Journal | project-notes.txt created in the Documents directory |
| 20:43:56.446 | USN Journal | project-notes.txt deleted |
| 20:44:34.998 | Sysmon / Prefetch / Memory | PowerShell initiated Notepad with update-check.ps1 |
| 21:03:03 | Volatile Memory | WinPmem memory acquisition process commenced |

### Timeline Interpretation

The correlated artifacts reconstruct a coherent sequence of activity.

The PowerShell script first appeared in the user's Downloads directory and was executed approximately 77 seconds later. Execution was followed by creation of script-generated files and establishment of a user-level Registry Run-key persistence mechanism.

A temporary text file was then created and deleted within approximately 15 seconds. Shortly afterwards, the downloaded PowerShell script was opened in Notepad.

The post-incident memory image preserved the Notepad process and its command line, providing volatile-memory corroboration of activity already identified through disk and event-log artifacts.

The timeline was constructed from multiple independent evidence sources rather than a single telemetry stream. This allowed individual observations to be cross-validated and reduced reliance on any one forensic artifact.

## 7. Limitations and Forensic Considerations

The following limitations were considered when interpreting the evidence:

### Controlled Simulation

This investigation was conducted in a controlled home-lab environment using benign activity. The observed behaviour was intentionally generated for forensic analysis and should not be interpreted as evidence of malware execution or a real compromise.

### Live Acquisition

Memory and endpoint artifacts were collected from the running system. Live acquisition necessarily interacts with the target and can introduce additional processes, filesystem activity, and other state changes.

A VMware snapshot of the post-incident state was created before forensic acquisition to preserve the lab system at that stage of the investigation.

### Memory Image Storage

Due to laboratory storage constraints, the memory image was written to the investigated system rather than to dedicated external forensic media. This is not the preferred approach for a formal forensic acquisition and was treated as a laboratory limitation.

### Logical Artifact Collection

KAPE was used to acquire selected forensic artifacts rather than a complete forensic disk image. Autopsy therefore examined the preserved logical collection and did not perform a full examination of the original disk or unallocated space.

### Deleted File Recovery

USN Journal records established that project-notes.txt was created, written to, and deleted. References were also identified in other artifacts, including the NTFS $LogFile and PowerShell history.

The corresponding filename was not available through the parsed MFT and the contents of the deleted file were not recovered. No claim is therefore made regarding recovery of its contents.

### Network Evidence

No packet capture was collected during the file transfer. Volatility netscan did not recover the earlier HTTP connection from the later memory image.

Network activity was reconstructed from endpoint evidence, including PowerShell command history, filesystem activity, and matching SHA-256 hashes of the source and downloaded artifacts.

### Timestamp Handling

The Windows system was configured for Europe/Dublin and was operating on Irish Standard Time (UTC+1) during the incident. Evidence sources were reviewed for their timestamp representation and significant events were normalized to UTC for timeline correlation.

### Tool Compatibility

The PECmd version bundled with the installed KAPE configuration did not support the newer Windows Prefetch format encountered during the investigation. A current standalone version of PECmd was used successfully to parse the artifacts.

### Autopsy Processing

Autopsy reported an ingest error while processing the Microsoft Edge WebCacheV01.dat artifact. Browser activity was not material to the investigation, and the error did not affect the evidence used to support the principal findings.

### Evidential Interpretation

Cryptographic hash comparison demonstrated that the downloaded PowerShell script was byte-for-byte identical to the controlled source artifact. This supports artifact integrity within the laboratory exercise but is not presented as a formal legal chain of custody.

## 8. Conclusion

The forensic examination successfully reconstructed the activity that occurred on WIN11-DFIR during the investigated period.

The evidence established that update-check.ps1 was transferred from the controlled test server at 192.168.134.128 to the dfiruser Downloads directory and subsequently executed through Windows PowerShell.

Following execution, filesystem artifacts associated with the script were created and a user-level persistence mechanism named UpdateCheck was established through the Windows Registry Run key. A temporary file, project-notes.txt, was later created, written to, and deleted. The downloaded PowerShell script was subsequently opened in Notepad.

These findings were not dependent on a single telemetry source. Significant events were independently corroborated across Sysmon, Prefetch, Registry data, NTFS metadata, the USN Journal, PowerShell command history, shortcut artifacts, and volatile memory.

Volatile-memory analysis additionally confirmed that Notepad remained active with update-check.ps1 supplied on its command line when memory was acquired. No historical HTTP connection associated with the earlier transfer was recoverable from the later memory image, and this negative result was retained as an investigative limitation rather than interpreted as evidence that the transfer had not occurred.

The investigation also demonstrated the importance of cross-artifact correlation. In particular, project-notes.txt was no longer represented by a recoverable filename in the parsed MFT, while its creation and deletion remained reconstructable through the USN Journal and were corroborated by other artifacts.

Overall, six principal findings were assessed as confirmed with high confidence. The investigation demonstrates a repeatable forensic workflow covering evidence preservation, volatile-memory acquisition, endpoint artifact collection, artifact parsing, timeline construction, cross-validation, interpretation of negative findings, and documentation of forensic limitations.




