# rdrleakdiag.exe Abuse

## Overview

`rdrleakdiag.exe`, is a diagnostic tool from SysinternalsSuite,that in the words of Google:
"a diagnostic tool developed by Microsoft Windows to troubleshoot memory leaks in the Redirected Drive Buffering Subsystem (Rdbss) driver"

The bottom line for the security enthusiasts reading this is not the usage of the tool, but the way to abuse it.
due to the nature of the tool that enables a user to perform debbuging actions on the memory, the tool is abused to enable some users (or attackers) to access the memory of some sensitive processes
or more specifically, the lucrative `lsass.exe` process that stores and manages credentials such as passwords, certificates, and security tokens in memory.
sounds easy to exploit? that because it actually is!

## Detection

The detection of this event is relatively simple as the exploitation, as the action generates four events (Assuming you use "Sysmon" for logging)
* The command execution:
  - The rule "rdrleakdiag_memory_dump_from_lsass" detects just that! the command that is executed goes as follows:
    `CommandLine: rdrleakdiag.exe  /p {PID} /o {path} /fullmemdmp`
    - The command specifies the "Process ID" of the desired process, this flag can be executed either as `-p` or `/p` **_Do not specify the process number as it is dynamic_**
    - The flag `/o` specifies the desired location of the output file. this flag is easily replaceable by `>` or `>>` and probably by other options as well.
    - The `/fullmemdmp` option specifies to generate a full memory dump of the desired process but it can also be replaced with options like `memdump` and such so this option should cover the options.
* The Remote Thread Creation:
  - The rule "rdrleakdiag_access_to_lsass" is set to detect this event.
    - The source and the target image illustrate the "SourceImage" `rdrleakdiag.exe` accessing the "TargetImage" `lsass.exe`.
* The sub-process creation:
  - There is no suggested rule to detect this behavior as it is recorded only as the process creation of the `lsass.exe` process by the `lsass.exe` process. with that said, it is still may be an important activity to log for your forensic analysis or with the use of most of the SIEMs, to create a nice _correlation rule_.
* The File Creation:
  - There is no suggested rule to detect this behavior as the file creation is a very dynamic event and so it is not suggested to create a rule based on it however it will definitely be usefull for forensics and as before - a nice _correlation rule_.

### Correlation

Although these rules are effective enough on their own, there is a possibility for a nice little correlation rule that may be helpful in determining if the attack succeeded, detecting the output file quickly after the incident and creating a more precise playboys and automation.

As described in the previous section, the correlation rule consists of four detections for the four events that accounts.

1. First, use the rule "rdrleakdiag_memory_dump_from_lsass" to detect the malicious command execution. the event will provide the following fields that may be useful for the correlation:
 - User
 - ProcessId
2. The second phase is to use the rule "rdrleakdiag_access_to_lsass" to detect the malicious thread creation. the event that accour should contain a "SourceProcessId" field that will match the ProcessId field from the previous event as they are sequential. with the following fields you can continue the correlation.
 - TargetProcessId
3. The third phase is detecting the execution of `lsass`. however, **you should not implement an offense rule based on this event**.  The sole purpose of this query will be for the correlation. this event will be matched based the "ParentProcessId" field and the "TargetProcessId" field in the previous event. the "ParentProcessId" will also be usefull for detecting the following event.
Here is how the log looks like, for creating the relevant query:
`Level	Date and Time	Source	Event ID	Task Category
Information	4/16/2020 3:47:36 PM	Microsoft-Windows-Sysmon	1	Process Create (rule: ProcessCreate)	"Process Create:
RuleName: 
UtcTime: 4/16/2020
ProcessGuid: {GUID}
ProcessId: {PID}
Image: C:\Windows\System32\lsass.exe
FileVersion: 10.0.19041.488 (WinBuild.160101.0800)
Description: Local Security Authority Process
Product: MicrosoftÂ® WindowsÂ® Operating System
Company: Microsoft Corporation
OriginalFileName: lsass.exe
CommandLine: C:\WINDOWS\system32\lsass.exe
CurrentDirectory: C:\WINDOWS\system32\
LogonId: 0x3E7
TerminalSessionId: 0
IntegrityLevel: System
Hashes: MD5= {Hash},SHA256={Hash},IMPHASH={Hash}
ParentProcessGuid: {guid}
ParentProcessId: 668
ParentImage: C:\Windows\System32\lsass.exe
ParentCommandLine: C:\WINDOWS\system32\lsass.exe
ParentUser: {User}`
4. the fourth and final phase of the correlation is detecting the file created. in that case also, **you should not implement an offense rule based on this event**. but you should definitely look at the event if all other events happened. the event will be connected to the previous events based on the "Image" and the time of the event.
Here is how the log looks like, for creating the relevant query:
`Level	Date and Time	Source	Event ID	Task Category
Information	9/28/2020 3:47:36 PM	Microsoft-Windows-Sysmon	11	File created (rule: FileCreate)	"File created:
RuleName: 
UtcTime: 4/16/2020
ProcessGuid: {GUID}
ProcessId: 3352
Image: C:\WINDOWS\system32\rdrleakdiag.exe
TargetFilename: C:\Users\wanwan\Desktop\minidump_668.dmp
CreationUtcTime: 4/16/2020
User: {User}`

## Contributing

Contributions to improve the accuracy and scope of these detection rules are welcome. Please fork the repository, make your changes, and submit a pull request with a clear description of the proposed updates.

## License

This project is licensed under the [MIT License](LICENSE).
