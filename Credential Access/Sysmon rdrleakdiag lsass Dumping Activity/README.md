# rdrleakdiag.exe Abuse

## Overview

`rdrleakdiag.exe`, is a diagnostic tool from SysinternalsSuite,that in the words of google:
"a diagnostic tool developed by Microsoft Windows to troubleshoot memory leaks in the Redirected Drive Buffering Subsystem (Rdbss) driver"

The bottom line for the security enthusiasts reading this is not the usage of the tool, but the way to abuse it.
due to the nature of the tool that enables a user to perform debbuging actions on the memory, the tool is abused to enable some users (or attackers) to access the memory of some sensitive processes
or more specifically, the lucrative `lsass.exe` process that stores and manages credentials such as passwords, certificates, and security tokens in memory.
sounds easy to exploit? that because it actually is!

## Detection

The detection of this event is relativly simple as the exploitation, as the action generates four events (Assuming you use "Sysmon" for logging)
* The command execution:
  - The rule "rdrleakdiag_memory_dump_from_lsass" detects just that! the command that is executed goes as follows:
    `CommandLine: rdrleakdiag.exe  /p {PID} /o {path} /fullmemdmp`
    - The commnad specifies the "Process ID" of the desired process, this flag can be executed either as `-p` or `/p` **_Do not specify the process number as it is dynamic_**
    - The flag `/o` specifies the desired location of the output file. this flag is easily replacable by `>` or `>>` and probably by other options as well.
    - The `/fullmemdmp` option specifies to generate a full memory dump of the desired process but it can also be replaced with options like `memdump` and such so this option should cover the options.
* The Remote Thread Creation:
  - The rule "rdrleakdiag_access_to_lsass" is set to tetect this event.
    - The source and the target image ilustrate the "SourceImage" `rdrleakdiag.exe` accessing the "TargetImage" `lsass.exe`.
* The sub-process creation:
  - There is no suggested rule to detect this behaviour as it is recorded only as the process creation of the `lsass.exe` process by the `lsass.exe` process. with that said, it is still may be an important activity to log for your forensic analysis or with the use of most of the SIEMs, to create a nice _corralation rule_.
* The File Creation:
  - There is no suggested rule to detect this behaviour as the file creation is a very dynamic event and so it is not suggested to create a rule based on it however it will definitly be usefull for forensics and as before - a nice _corralation rule_.

## Purpose

These rules are designed to detect potential misuse or unauthorized use of `rdrleakdiag.exe` against sensitive system processes like `lsass.exe`, which could indicate credential access or other malicious activities.

## Usage

Ensure these detection rules are implemented in a security monitoring system that supports Windows Sysmon logs. Adjust field mappings and thresholds as per your environment's requirements to minimize false positives and effectively detect suspicious activities.

## Contributing

Contributions to improve the accuracy and scope of these detection rules are welcome. Please fork the repository, make your changes, and submit a pull request with a clear description of the proposed updates.

## License

This project is licensed under the [MIT License](LICENSE).
