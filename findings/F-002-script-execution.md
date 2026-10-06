**Finding ID:** F-002  
**Title:** PowerShell Script Execution  
**Assessment:** Confirmed  
**Confidence:** High

## Finding
The downloaded update-check.ps1 script was executed by the dfiruser account using Windows PowerShell.

## Evidence
- Sysmon records powershell.exe at 2026-10-05 20:42:30.117 UTC with the command line:
  powershell.exe -NoProfile -ExecutionPolicy Bypass -File C:\Users\dfiruser\Downloads\update-check.ps1
- The process ran as WIN11-DFIR\dfiruser with Medium integrity.
- Prefetch records POWERSHELL.EXE execution at approximately 20:42:30 UTC.
- The USN Journal records update.log and execution-time.txt being created and written under:
  C:\Users\dfiruser\AppData\Local\Microsoft\UpdateCache
  immediately after the PowerShell execution.
- PowerShell ConsoleHost history contains the corresponding script execution command.

## Interpretation
Independent process-execution, Prefetch, command-history, and filesystem artifacts establish that update-check.ps1 was executed successfully. The creation of update.log and execution-time.txt immediately after process creation is consistent with the known behaviour of the controlled script.

## Limitations
Prefetch establishes executable activity but does not independently provide the complete PowerShell command line. Command-line attribution is therefore primarily supported by Sysmon and corroborated by PowerShell history. The contents of the execution-created files were not recovered from the KAPE logical collection.


