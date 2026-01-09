---
layout: striped-default
title: "When Computer Says Yes (automatically): Understanding UAC Auto-Elevation – Part 1/2"
date: 2025-09-02
tags: [blue-team, uac]
---

Windows systems have long wrestled with the balance between usability and security; we joke about how every other version of Windows is "the bad version", when security takes precedence over user-experience (I'm talking to you, Windows Vista!).
One of Microsoft's key mechanisms for keeping users safe – while still allowing administrators to perform their job – is **User Account Control (UAC)**; but like many security features, the implementation leaves certain "cracks" that attackers can exploit. One of those cracks lies in the concept of **auto-elevated executables**.<br><br>
<!--more-->


## What is UAC and why do we need it?

It's sad to admit, but before Windows Vista (back in 2007), most users ran their daily tasks with <ins>full administrative rights</ins>. And as any cyber-security person would tell you: if one has admin rights on a machine (whether it be malware, or just a careless user) – they can wreak havoc at the system level.
To address this issue, Microsoft introduced **User Account Control (UAC)**. The principle behind it was simple: even if you're logged in as an administrator, applications start in a standard user context (**medium-integrity**).

**Integrity Levels** in Windows are part of _Mandatory Integrity Control_ (MIC), which decides how processes interact with objects. The common levels are:

- **Low** → e.g., Internet Explorer Protected Mode, Edge/Chrome sandboxes
- **Medium** →  the default for processes launched by regular user accounts
- **High** → for elevated processes (admin with UAC approval)
- **System** → for core Windows services

When an action requires elevated rights (such as installing software, modifying system files or changing drivers), Windows prompts you with the familiar UAC consent box: __"Do you want to allow this app to make changes to your device?"__
<img src="/assets-striped/posts/2025-09-02-when-computer-says-yes-uac-auto-elevation-part-1/consent.png" alt="UAC consent box" class="center-image">

This forced boundary between normal tasks and admin-level actions dramatically reduced the success of many malware strains, which would otherwise run freely and uninterrupted. In essence: UAC was designed to stop "silent" privilege escalations. <br>
That said, UAC is not considered a "security barrier" by Microsoft.

<br>

## What Is Auto-Elevation?

Here’s where usability fights back!
Certain Windows components are considered so essential and trustworthy that Microsoft decided they should auto-elevate; meaning – they should run with administrator privileges  **without prompting the user**. <br>
Examples of such components include:

- `eventvwr.exe` (Event Viewer)
- `mmc.exe`      (Microsoft Management Console)
- `compmgmt.msc` (Computer Management)

These executables are marked in their application manifest(*) with the flag _**autoElevate="true"**_. When Windows sees this flag and confirms the binary is signed by Microsoft, it silently runs the process in a high-integrity (administrator) context – but only if the user is a member of the local Administrators group.

> (*) `An application manifest is a small XML file, either embedded inside an executable or provided alongside it, that tells Windows important metadata about the application – such as required privileges, compatibility settings, DPI awareness and whether it should auto-elevate when run.`

<br>

### This sounds cool. Does that mean a standard ("non-admin") user can use a UAC bypass in order to get local admin privileges?
In one word: **no**. A standard user (medium-integrity) <ins>cannot</ins> use a UAC bypass in order to elevate themselves to admin (high-integrity); UAC auto-elevation only work if the user was already a member of the local Administrators group. <br>
**Why is that?** – this has to do with Windows' *split-token model**.

- When a **standard user** (not in Administrators group) logs into a Windows machine, they only have a standard token; that token gives access to resources with medium-integrity (or lower). They do not have a high-integrity token at all – there’s nothing to "bypass" into.
Therefore: no UAC bypass technique can elevate them to high-integrity.
- When an **Administrator user** (in Administrators group) logs into a Windows machine, Windows creates at logon-time two tokens for them: <br>
1) A filtered token (medium-integrity) → used for normal processes. <br>
2) A full admin token (high-integrity) → locked behind UAC approval. <br>
By default, everything runs under the filtered token. To do something privileged, the user either:
	- Approves a UAC prompt (consent or credentials), which switches the process to the high-integrity token, **or**
	- A vulnerable/poorly designed auto-elevating process silently uses the full token without prompting.
<br><br>

## How to Find Auto-Elevated Files
If you’re an attacker, a defender or a researcher, you might want to know which executables in your environment are set to auto-elevate. Here are a few approaches:
- **Check application manifests**
Executable files often contain an embedded XML manifest. Tools such as **"sigcheck"** or **"Resource Hacker"** can parse these manifests to look for the _"autoElevate"_ flag.
- **Use Community Tools**
Security researchers have released scripts that enumerate all binaries on disk and flag the ones with _autoElevate="true"_. <br>
If you like coding, you could even write code yourself (using a language such as Python or Powershell) to build a simple program that searches for this type of files.
- **Refer to public lists**
Certain penetration testers and red teams maintain curated lists of Windows **LOLBins** ("Living Off the Land Binaries"), including auto-elevated executables.
<br>

Check out my [Github Repository](https://github.com/Sam0rai/uac_auto_elevate_finder) (a simple Rust program which recursively iterates over a designated directory and searches for auto-elevated executables).

**Next:** [Part 2 – deeper dive into exploitation and mitigation](/blog/2025/09/03/when-computer-says-yes-uac-auto-elevation-part-2/)
