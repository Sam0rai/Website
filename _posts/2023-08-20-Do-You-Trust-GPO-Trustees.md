---
layout: striped-default
title: "Do You Trust GPO Trustees?"
date: 2023-08-20
tags: [active-directory, blue-team, red-team]
---
## Introduction
Group Policy Objects (or GPOs) are a valuable and necessary pillar of any Windows-based organization. They enforce various rules and definitions on hosts, and allow sysadmins to govern the Active Directory domain, on all its users, endpoints and servers. <br>
In the ever-evolving landscape of cybersecurity, GPOs have not gone unnoticed. One notable technique in penetration testing involves the abuse of GPOs by exploiting overly permissive permissions pertaining to them. <br>
This blog post discusses the issue, and introduces a tool designed to assist both Red and Blue Teams in identifying such misconfigurations within GPOs.
<!--more-->

## The Exploitative Potential of GPOs
GPOs serve as a vital component of Windows environments, enabling centralized management of security and other settings across a network. Unfortunately, their potency can also be harnessed for malicious purposes. Red Teamers have been leveraging GPOs with excessively broad permissions for a long time now, altering them to achieve their goals, such as by inserting malicious commands or scripts, which can then be executed on various endpoints within the targeted organization. This manipulation can yield a range of threats, including remote code execution, privilege escalation, persistence and more. <br>

The source of the problem lies with the permissions set on the manipulation of a Group Policy Object by a user (also called a "Trustee” by Microsoft in this context); while any user\trustee in the domain should (in most cases) have a read permission for a GPO, only a specific set of users or AD groups should have permissions to alter said GPO – mainly the Domain Controller’s "SYSTEM” account and users in the "Domain Admins” and "Enterprise Admins” AD groups. I refer to those as "default trustees”. <br>

One such tool to assist Red Teamers with this kind of GPO exploitation is FSecure LABS’ [SharpGPOAbuse](https://github.com/FSecureLABS/SharpGPOAbuse). Given a vulnerable GPO which holds permissions so that it can be changed by a non-default trustee (i.e.: a user who’s not an Enterprise or Domain Admin), and given the Red Team can impersonate that trustee – they now have the power to alter the GPO, and thus control in some fashion all the hosts which are enforced by that GPO.

## Finding Vulnerable GPOs
 <br>
<figure class="image centered">
  <img src="/assets-striped/posts/2023-08-20-Do-You-Trust-GPO-Trustees/Why-Not-Both.jpg" alt="Velociraptor Private Detective">
  <figcaption>Red-Team or Blue-Team tool?</figcaption>
</figure>

To counteract this potential for abuse, I have developed a Powershell script I’ve simply named:
[Find-GPO-Non-Default-Trustees](https://github.com/Sam0rai/Blueteam-Tools/). Whether you’re a Red Teamer simulating attacks or a member of the Blue Team fortifying defenses – this tool is for you! <br>
**What does it do?** <br>
The script performs an enumeration of all GPOs within a designated AD Domain. For each GPO, it checks for non-default trustees (users or AD groups) with the capability to modify or delete the respective GPOs. The following permissions are checked: **"GpoEditDeleteModifySecurity", "GpoEditDeleteModify", "GpoEdit", "GpoDelete", "GpoCreate", "GpoModifySecurity"**.
This list provides essential insights, outlining the GPO name alongside the associated user or AD group, as well as their corresponding permissions.
It is also worth noting that the tool can be run even by a <ins>low-privileged</ins> user!

## Conclusion
As the cybersecurity landscape continues to evolve, it is imperative to stay ahead of potential threats. The misuse of GPOs for unauthorized access and compromise represents a genuine concern. <br>
By shedding light on non-default trustee abuses, this tool equips Blue Teams with the means to strengthen their organization’s defense mechanisms, while also enabling Red Teams to find such misconfigurations and exploit them responsibly.