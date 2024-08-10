# LSASS Memory Access

## Overview

Accessing `lsass.exe` memory is similar to playing the lottery because a hacker risks detection for credentials of unknown value. This process is one of the designated processes for handling security-related tasks in Windows, such as enforcing policies and **handling credentials**. With its full name _"Local Security Authority Subsystem Service"_, it is a lucrative target for attack.

## Detection

Although LSASS access generates only two primary events (a handle request and the LSASS access event), it can be relatively tricky to determine which access mask an attacker will need for dumping the memory.

### The Handle Request

- **Rule: "A Handle to LSASS Was Requested"**
  - This rule detects requests for the needed permissions to access the LSASS process:
    - **`Object Name: lsass.exe`**
      - The object name corresponds to the target for which the handle was requested, and it is the lucrative **"LSASS"** process.
    - **`Process Name: cscript.exe`**
      - The process name detected in the rule should be **cscript.exe**. This executable runs scripts automatically or via the command line. Access to the LSASS process is often made by scripts like "WinPEAS". __Note that tools such as WinPEAS may have an executable version; monitoring the executable's name is crucial, although attackers might rename these tools to evade detection.__
    - **Access Masks to Monitor:**
      - **0x10 (PROCESS_VM_READ):** Allows reading from process memory, crucial for credential dumping.
      - **0x0410 (PROCESS_QUERY_INFORMATION + PROCESS_VM_READ):** Combines querying process information with reading memory, essential for extracting credentials.
      - **0x140:** Includes permissions for creating threads and querying information, which might be used in attacks.
      - **0x1FFFFF (Full Control):** Provides comprehensive access to the process, including reading, writing, and modifying process attributes.
      - **0x1F0FFF (Extensive Control):** Similar to full control but slightly less inclusive, still allows significant access.
      - **0x1F3FFF (Similar to Full Control):** A variant of full control that includes a wide range of permissions.
      - **0x1438:** Includes permissions for process duplication and creating threads, useful for advanced manipulation.
      - **0x0F10:** Combines read and write access, essential for both extracting and modifying process memory.
      - **0xC0000 (WRITE_DAC + WRITE_OWNER):** Allows modifying security attributes and ownership, potentially used to alter process security settings.
      - **0x4000 (PROCESS_VM_OPERATION):** Allows performing various operations on the process's memory, relevant for attacks involving memory manipulation.
      - **0x0008 (WRITE_DAC):** Allows modifying the discretionary access control list, useful for altering permissions.
      - **0x0010 (WRITE_OWNER):** Allows changing ownership of the process, useful for privilege escalation.
      - **0x0004 (READ_CONTROL):** Allows querying the security information, useful for understanding and manipulating access controls.

### Access to LSASS Process Memory

- **Rule: "Access to LSASS Process Memory"**
  - This rule is built similarly to the previous rule but differs in event ID, as it continues from the handle request event. After obtaining the handles, they should be used for access.

### Correlation

There is not much to correlate; these events can be detected individually or in chronological order: first the handle request, followed by the access to LSASS process memory, to indicate a successful exploitation.

## Response

The response should address the following questions:
- Was the attack successful?
- What user ran the script?
- What credentials might have been obtained?
- What access vector did the attacker use to access the computer?
- Did the attacker attempt to log in as a different user or move laterally?

To respond to this incident, you should try to prevent the attacker from moving or exporting the credentials. You should also start investigating the initial access vector to ensure you understand what path the attacker might take to use the obtained credentials.

## Contributing

If you have any inputs to give me, mistakes you've found, suggestions, questions, or anything else, I would like to hear from you about it.
