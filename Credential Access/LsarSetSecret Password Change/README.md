# LsarSetSecret Machine Account Password Change Attempt

## Overview

`LsarSetSecret`, is a function that helps to manage the LSA which maintains the registery parts that store sensitive information like keys and hashed passwords.
This function allowes to set the value of a secret object stored by the LSA (Local Security Authority)

For the security people around here this brobably already sounds alarming. basically, abusing this function will grant you the ability to dump or change secrets such as:
- Machine account passwords
- Service account passwords
- Other sensitive information needed by the system or specific applications
The eploitation of this function can be performed by a schedueled task, Malicious Services and malwares, such as with the use of mimikatz `lsadump::secrets` command.

## Detection

As the vector of the exploitation can vary, we will try to detect not the action of exploitation but the result. the detection of the `LsarSetSecret` function abuse should be based on the two seperate events of registery key creation and the setting of the value in the registery (event IDs 12-13)
* The key Creation:
  - The rule "LsarSetSecret_key_creation" will detect any creation of keys in the 
    `HKLM\SECURITY\Policy\Secrets\$MACHINE.ACC\CurrVal` registry path.
    - The action will most likly be performed by lsass as it has the privileges to manage the LSA throught these functions and it is injectable and overall a highly targeted service by tools such as mimikatz and more.
      - The first option to detect this event is as described in the rule. this may be more precise way of detection that will eliminate alot of false positives. this means monitoring the abuse of this function by lsass.
      - The second option to detect this event is using a broader detection method by monitoring any change done using this function by any initiator (Image). **This can couse the rule to detect alot of False Positive events so be carefull when implementing these rules in your environment**
      - Also it is advised to monitor all the actions done by the function and currate a whitelist of permitted initiators.
* The Setting of The New Value
  - The rule "LsarSetSecret_value_set" will the detect the setting of the new passord or as it is named "the value inpoted in to the registery" in the:
    `HKLM\SECURITY\Policy\Secrets\$MACHINE.ACC\CurrVal` registry path.
    - This event is a proggression of the "LsarSetSecret_key_creation" as it will detect the follow-up action done by the system.
    - the options of detection with this rule can be as described in the previus section.


### Correlation

There is no correlation that i can think of other then correlating these two rules basing the corellation on the "ProcessId" of the two events to know if the action Succeeded.

## Response

The response should be the examination of the process that invoked the dangerous function or the lsass process, depends on what you chose to monitor

the investigation needs to answer the following questions:
* Who is the initiator of the action?
* What made the initiator to use the dangerous function?
* What data was changed by the function?

You should act quickly to understand what privileges does the attacker has and on which user and also analyze the data changed in the registery and try understanding to which users the attacker might have gained access with this exploit.

## Contributing

Contributions to improve the accuracy and scope of these detection rules are welcome. Please fork the repository, make your changes, and submit a pull request with a clear description of the proposed updates.
