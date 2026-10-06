**Finding ID:** F-001  
**Title:** PowerShell Script Acquisition  
**Assessment:** Confirmed  
**Confidence:** High

## Finding
The file update-check.ps1 was created in the Downloads directory of the dfiruser account during the incident window.

## Evidence
- USN Journal records update-check.ps1 FileCreate at 2026-10-05 20:41:12.9858330 UTC, followed by DataExtend and Close activity.
- PowerShell ConsoleHost history records an Invoke-WebRequest request for http://192.168.134.128:8080/update-check.ps1 with the destination set to the dfiruser Downloads directory.
- SHA-256 comparison performed during the controlled simulation showed the downloaded file matched the source artifact.

## Interpretation
The evidence supports that update-check.ps1 was transferred from the controlled test server at 192.168.134.128 to the WIN11-DFIR workstation.

## Limitations
PowerShell history records a command entered by the user and does not by itself prove successful execution. The successful transfer is corroborated by the filesystem artifact and matching file hash.


