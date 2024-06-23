# Detection Rules for rdrleakdiag.exe Abuse

## Overview

This repository contains detection rules for potential abuse scenarios involving `rdrleakdiag.exe`, a diagnostic tool from SysinternalsSuite, attempting unauthorized actions on `lsass.exe` memory or performing memory dumps.

### Rules Included

1. **Title**: rdrleakdiag.exe Access to lsass
   - **ID**: e0393bbf-cced-4f70-824f-5d8fed343891
   - **Description**: Detection of rdrleakdiag.exe accessing lsass.exe memory.
   - **Author**: Michael Vilshin
   - **Date**: 06/23/2024
   - **Log Source**: Windows Sysmon (process_creation)
   - **Detection Criteria**:
     - EventID: 8
     - SourceImage ends with `\rdrleakdiag.exe`
     - TargetImage ends with `\lsass.exe`
   - **Fields**: EventID, UtcTime, SourceProcessId, SourceImage, TargetProcessGuid, TargetProcessId, TargetImage, NewThreadId, StartModule, SourceUser, TargetUser
   - **False Positives**: System Administration Team uses diagnostics and debugging tools.
   - **Level**: High
   - **References**: [LOLBAS - Rdrleakdiag](https://lolbas-project.github.io/lolbas/Binaries/Rdrleakdiag/)
   - **Tags**: attack.credential_access, attack.t1003.001

2. **Title**: rdrleakdiag.exe Memory Dump From Lsass
   - **ID**: 6addb3fb-c640-4b31-8147-418bf6e920ff
   - **Description**: Attempted memory dump by rdrleakdiag.exe from lsass.exe.
   - **Author**: Michael Vilshin
   - **Date**: 06/23/2024
   - **Log Source**: Windows Sysmon (process_creation)
   - **Detection Criteria**:
     - EventID: 1
     - Image ends with `\rdrleakdiag.exe` OR OriginalFileName is 'RdrLeakDiag.exe'
     - CommandLine contains 'rdrleakdiag.exe' and '/p' or '-p'
     - CommandLine contains 'memdmp' or 'fullmemdmp'
   - **Fields**: UtcTime, ProcessId, Image, OriginalFileName, CommandLine, CurrentDirectory, User, LogonId, TerminalSessionId, Hashes, ParentProcessId, ParentImage, ParentCommandLine, ParentUser
   - **False Positives**: System Administration Team uses diagnostics and debugging tools.
   - **Level**: High
   - **Tags**: attack.credential_access, attack.t1003.001

## Purpose

These rules are designed to detect potential misuse or unauthorized use of `rdrleakdiag.exe` against sensitive system processes like `lsass.exe`, which could indicate credential access or other malicious activities.

## Usage

Ensure these detection rules are implemented in a security monitoring system that supports Windows Sysmon logs. Adjust field mappings and thresholds as per your environment's requirements to minimize false positives and effectively detect suspicious activities.

## Contributing

Contributions to improve the accuracy and scope of these detection rules are welcome. Please fork the repository, make your changes, and submit a pull request with a clear description of the proposed updates.

## License

This project is licensed under the [MIT License](LICENSE).
