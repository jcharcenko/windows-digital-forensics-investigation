**Finding ID:** F-006  
**Title:** Controlled Network Transfer  
**Assessment:** Confirmed  
**Confidence:** High

## Finding
The WIN11-DFIR workstation retrieved update-check.ps1 from the controlled test server at 192.168.134.128 over TCP port 8080.

## Evidence
- PowerShell ConsoleHost history records:
  Invoke-WebRequest -Uri "http://192.168.134.128:8080/update-check.ps1" -OutFile "$env:USERPROFILE\Downloads\update-check.ps1"
- The USN Journal records update-check.ps1 being created and written in the dfiruser Downloads directory immediately after the recorded request.
- SHA-256 comparison performed during the controlled simulation showed that the downloaded artifact matched the source artifact hosted on the controlled test server.
- Autopsy keyword analysis independently identified references to 192.168.134.128 in preserved endpoint artifacts.

## Volatile Network Analysis
Volatility windows.netscan returned no network connections from the acquired memory image. Memory acquisition occurred approximately 22 minutes after the file transfer, so the absence of the earlier HTTP connection is consistent with a short-lived connection no longer being resident or recoverable at acquisition time.

## Interpretation
The command history, filesystem evidence, and matching artifact hashes collectively support successful transfer of update-check.ps1 from the controlled server. The negative netscan result does not contradict this finding because it represents network state recoverable from memory at the later acquisition time rather than a historical record of all prior connections.

## Limitations
No packet capture was collected for the transfer, and no surviving network socket attributable to the HTTP request was recovered from memory. The network interaction is therefore reconstructed from endpoint artifacts rather than packet-level evidence.


