---
layout: striped-default
title: "Hunting For Saved Credentials in WinSCP and MobaXterm"
date: 2023-02-02
---
## Introduction
When attackers get a foothold on a compromised machine, one of their first courses of action is seeking means for elevation of privileges; that hopefully allows them access to other resources in the target organization, a way to move laterally and reach their goals. <br>
One way for privilege escalation is by extracting passwords from areas they reside in. This could be as easy a task as opening a "passwords.txt" file located on the user's Desktop, or as "hard" as dumping credentials from the machine's volatile memory.
<!--more-->
<br><br>
Users in organizations use various tools to authenticate to servers, databases, applications and so forth, under a variety of protocols (such as SSH, FTP, RDP). These tools usually allow the enduser to save their credentials to login to systems; the problem is these tools store the credentials locally on the machine, sometimes in cleartext, and other times encoded (yet not encrypted), or encrypted – but in a way they can be easily decrypted by an attacker. <br>
Researchers from [XMCyber](https://www.xmcyber.com/) have recently released a blog post about this issue, titled ["Extracting Encrypted Credentials from Common Tools"](https://www.xmcyber.com/blog/extracting-encrypted-credentials-from-common-tools-2/). The post elaborates on where the credentials are stored in 3 common tools: WinSCP, MobaXterm and RoboMongo, and how they can be extracted and decoded or decrypted.
This following paragraphs will tackle this issue from a Threat-Hunting and Blue-Team perspective.

## Hunting For Stored Credentials
Enter [Velociraptor](https://docs.velociraptor.app/).
<div class="text-with-image">
    One could approach the task of searching for stored credentials in the aforementioned tools in various ways, using software inventory tools, EDR\XDR products, or even basic scripting.
    My personal favorite is an open-source threat-hunting platform named "Velociraptor". This platform is comprised out of agents installed on endpoints, and a main management server (from which hunts are performed). <br>
    The strength of Velociraptor lies in its versatile query language (called "VQL"); one can write a query and search for results across the entire organizational domain; here's a query I've written which searches an endpoint for any passwords saved in Registry by WinSCP:
  <img src="/assets-striped/posts/2023-02-02-Hunting-For-Saved-Credentials-in-WinSCP-and-MobaXterm/Velociraptor-Logo.png" alt="Velociraptor Logo" class="img-20">
</div>
```sql
Let SearchRegistryGlob = "HKEY_USERS\\S-1-5-21-\\Software\Martin Prikryl\\WinSCP 2\\Sessions\\*\\password"
SELECT Data.value as ObfuscatedPassword, FullPath, ModTime
FROM glob(globs=SearchRegistryGlob, accessor='reg')
```
And here's another query for searching for saved credentials in **MobaXterm**: <br>
```sql
Let SearchRegistryGlob = "HKEY_USERS\\S-1-5-21-*\\SOFTWARE\\Mobatek\MobaXterm\\{M,P,C}\\**"

SELECT Data.value as EncryptedCreds, FullPath, ModTime
FROM glob(globs=SearchRegistryGlob, accessor='reg')
```
The results for each query can then be saved to a csv file to be analyzed.

## Mitigations
<br>
### Blocking saving passwords in WinSCP
We can block the action of saving passwords in WinSCP by making the [following modifications](https://winscp.net/eng/docs/administration#configuring) to the registry: <br>
```text
[HKEY_LOCAL_MACHINE\SOFTWARE\Martin Prikryl\WinSCP 2]
"DisableOpenEdit"=dword:00000001
"DisablePasswordStoring"=dword:00000001
"DisableAcceptingHostKeys"=dword:00000001
```
<br>
### Blocking saving passwords in MobaXterm
We can do the same for MobaXterm by making the following modifications to the registry:
```text
[HKEY_CURRENT_USER\Software\Mobatek\MobaXterm]
"disable_password_saving"=dword:00000001
```
To block across the entire domain – the above mitigations can be defined in an Active Directory Group Policy (GPO).

<br>
### Monitoring
Using [Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon), we can define an audit log to be created any time a user succeeds storing credentials in any of the discussed tools (event IDs 12-14). These logs can then be sent to a SIEM, where rules can be created to alert whenever a user does manage to store their credentials in those tools. <br>
Here's the minimized Sysmon config file to enable such auditing:
```xml
<Sysmon schemaversion="4.50">
  <HashAlgorithms>md5,sha256,IMPHASH</HashAlgorithms>
  <EventFiltering>
  <!--SYSMON EVENT ID 12 & 13 & 14 : REGISTRY MODIFICATION [RegistryEvent]-->
  <RuleGroup name="" groupRelation="or">
    <RegistryEvent onmatch="include">
      <TargetObject condition="begin with">HKCU\Software\Mobatek\MobaXterm</TargetObject>
      <TargetObject condition="begin with">HKCU\Software\Martin Prikryl\WinSCP 2\Sessions\*\</TargetObject>
    </RegistryEvent>
  </RuleGroup>
  </EventFiltering>
</Sysmon>
```
<br>
In my org, I've witnessed 2 types of employees: "regular" users and IT users. <br>
The regular users used their day-to-day credentials to authenticate to designated servers, and thus don't need to store them in the aforementioned tools. <br>
IT users used specific "applicative" users (as opposed to their personal user-credentials) to login to certain system. A preferred solution for these types of users would be to save their credentials in a vault, or better still – use a **PSM** (= Privileged Session Manager) solution to login to servers.