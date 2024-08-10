# LsarSetSecret Machine Account Password Change Attempt

## Overview

`LsarSetSecret` is a function that helps to manage the LSA, which maintains the registry parts that store sensitive information like keys and hashed passwords. This function allows setting the value of a secret object stored by the LSA (Local Security Authority).

For security professionals, this probably already sounds alarming. Abusing this function grants the ability to dump or change secrets such as:
- Machine account passwords
- Service account passwords
- Other sensitive information needed by the system or specific applications

The exploitation of this function can be performed by scheduled tasks, malicious services, and malware, such as with the use of Mimikatz's `lsadump::secrets` command.

## Detection

As the vector of the exploitation can vary, we will try to detect not the action of exploitation but the result. The detection of the `LsarSetSecret` function abuse should be based on the two separate events of registry key creation and the setting of the value in the registry (event IDs 12-13).

### Key Creation

- The rule "LsarSetSecret_key_creation" will detect any creation of keys in the `HKLM\SECURITY\Policy\Secrets\$MACHINE.ACC\CurrVal` registry path.
  - The action will most likely be performed by `lsass.exe` as it has the privileges to manage the LSA through these functions and it is injectable and overall a highly targeted service by tools such as Mimikatz and more.
    - The first option to detect this event is as described in the rule. This may be a more precise way of detection that will eliminate a lot of false positives. This means monitoring the abuse of this function by `lsass.exe`.
    - The second option to detect this event is using a broader detection method by monitoring any change done using this function by any initiator (Image). **This can cause the rule to detect a lot of false positives, so be careful when implementing these rules in your environment.**
    - It is also advised to monitor all the actions done by the function and curate a whitelist of permitted initiators.

### Setting of the New Value

- The rule "LsarSetSecret_value_set" will detect the setting of the new password, or as it is named, "the value inputted into the registry" in the `HKLM\SECURITY\Policy\Secrets\$MACHINE.ACC\CurrVal` registry path.
  - This event is a progression of the "LsarSetSecret_key_creation" as it will detect the follow-up action done by the system.
  - The options of detection with this rule can be as described in the previous section.

## Correlation

There is no correlation that I can think of other than correlating these two rules based on the "ProcessId" of the two events to know if the action succeeded.

## Response

The response should involve examining the process that invoked the dangerous function or the `lsass.exe` process, depending on what you chose to monitor.

The investigation needs to answer the following questions:
* Who is the initiator of the action?
* What caused the initiator to use the dangerous function?
* What data was changed by the function?

You should act quickly to understand what privileges the attacker has and on which user. Also, analyze the data changed in the registry and try to understand which users the attacker might have gained access to with this exploit.

## Contributing

If you have any inputs to give me, mistakes you've found, suggestions, questions, or anything else, I would like to hear from you about it.
