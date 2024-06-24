# rdrleakdiag.exe Abuse

## Overview

`rdrleakdiag.exe`, is a diagnostic tool from SysinternalsSuite,that in the words of google:
"a diagnostic tool developed by Microsoft Windows to troubleshoot memory leaks in the Redirected Drive Buffering Subsystem (Rdbss) driver"

The bottom line for the security enthusiasts reading this is not the usage of the tool, but the way to abuse it.
due to the nature of the tool that enables a user to perform debbuging actions on the memory, the tool is abused to enable some users (or attackers) to access the memory of some sensitive processes
or more specifically, the lucrative `lsass.exe` process that stores and manages credentials such as passwords, certificates, and security tokens in memory.
sounds easy to exploit? that because it actually is!

## Purpose

These rules are designed to detect potential misuse or unauthorized use of `rdrleakdiag.exe` against sensitive system processes like `lsass.exe`, which could indicate credential access or other malicious activities.

## Usage

Ensure these detection rules are implemented in a security monitoring system that supports Windows Sysmon logs. Adjust field mappings and thresholds as per your environment's requirements to minimize false positives and effectively detect suspicious activities.

## Contributing

Contributions to improve the accuracy and scope of these detection rules are welcome. Please fork the repository, make your changes, and submit a pull request with a clear description of the proposed updates.

## License

This project is licensed under the [MIT License](LICENSE).
