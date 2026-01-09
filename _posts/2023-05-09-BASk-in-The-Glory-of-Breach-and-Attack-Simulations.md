---
layout: striped-default
title: "BASk in The Glory of Breach & Attack Simulations?"
date: 2023-05-09
tags: [blue-team, red-team]
---

## Introduction
One of the most popular (and “sexy”) fields in cyber-security is the field of penetration testing (or “pen-testing” \ “PT” for short); it is also a crucial component of any organization’s cybersecurity strategy.
In simple terms, pen-testing involves simulating an attack on a system or network, in order to identify potential vulnerabilities, misconfigurations and other weaknesses. By doing so, organizations can identify (in a safe manner) areas of their security infrastructure that need improvement – before an actual attacker exploits them.
<br>
As new technologies emerge, so do new terms come into existence. The notion that much of the work a human pen-tester does could be automated by a machine has created a new line of products, which Gartner has coined the term for as **B.A.S.**, or _“Breach & Attack Simulation”_ systems.
<!--more-->
<br>
These systems simulate attackers in various stages, and can perform operations such as initial access, privilege escalation and lateral movement, exfiltration and so forth, thus exposing gaps in the organizational security posture.

## Attacking The Network and The Zombie Apocalypse
I categorize a BAS product’s purpose into two main categories:

**1. Attacker in The Network** <br>
In this category, we define a scenario of an attacker residing inside the network. We choose a source the attacker has compromised (such as a host, or a certain user’s credentials), and we choose a destination they must reach. We then let the product strategize and seek out an attack path to fulfill its goal.
<br>
For example:
<figure class="image centered">
  <img src="/assets-striped/posts/2023-05-09-BASk-in-The-Glory-of-Breach-and-Attack-Simulations/01-Attacker-in-the-network.png" alt="Attacker in the Network">
</figure>
Our starting point is that the attacker has compromised **HostA**. They manage to retrieve the credentials for the local administrator account. With those credentials, they move laterally and compromise **HostB**. They proceed to extract from that host cached credentials of **Victim1** for the web server, thus allowing for further lateral movement in the network.

The “Attacker in The Network” scenario is one where we assume there could be some attack path between the chosen starting point (the initial compromised host), and the destination goal (the web server). So we define a starting point and an ending point – and let the product show us how an attacker would have compromised the web server.

We are not surprised that this was feasible, and now it’s up to us as the security-team to find a solution to prevent this attack path from happening again (for example: breaking the attack chain from **hostA** to **hostB** by implementing LAPS).
<br><br>
**2. The Zombie Apocalypse** <br>
Zombie apocalypse movies are filled with imagery of thousands of zombies storming gates and blockade walls, rushing to get to the other side.
<figure class="image centered">
  <img src="/assets-striped/posts/2023-05-09-BASk-in-The-Glory-of-Breach-and-Attack-Simulations/02-World-War-Z.png" alt="World War Z">
  <figacaption>Credit: The movie “World War Z”</figacaption>
</figure>
In this category, there are two networks that should never ever (ever!) have an attack path between them; for example: a DMZ network and an internal administrative network, or a critical network (where all organizational finance systems are located).
<figure class="image centered">
  <img src="/assets-striped/posts/2023-05-09-BASk-in-The-Glory-of-Breach-and-Attack-Simulations/03-Intranet-and-DMZ-diagram.jpg" alt="World War Z">
  <figacaption>Internal and external networks in an organization</figacaption>
</figure>
In this type of scenario, every single host and user in the DMZ network could be defined as being compromised (i.e.: a starting point); proverbial zombies trying to barge into the internal network, as it were. <br>
Reaching any host within an internal network from the DMZ should never happen – so if the BAS product does manage to find an attack path – I’d like to get an immediate alert on the matter (via email, SMS, alert in the SIEM, etc.).

## The Terminator Effect
We are all too familiar with the usage of the **APT** acronym (=Advanced Persistent Threat) in cyber-security, aren’t we? <br>
While there are different kinds of products under the BAS category, each with its own nuances and emphasis, they all market themselves as emulating an **APT**. Though some may beg to differ the meaning of “Advanced” in this context, they do share the “Persistent” and “Threat” pillars of an APT; by that I mean that these products simulate a **threat** (exploit known vulnerabilities and misconfigurations, extract credentials from memory, and so on), and they are **persistent**.
<br>
I refer to the persistence aspect as “The Terminator Effect”; to paraphrase a quote from the movie “The Terminator”:
> _“The attacker doesn’t feel any sympathy or regret and they’re not afraid. they’re a monster and they will not stop until they breach you”._

<br>
**Why is this so important?** <br>
A security consultancy firm may be requested to perform a PT on a client’s network, and a human pen-tester would be commissioned to work for two days, or a week, or two weeks at a client’s site, trying their best to achieve some predefined goal. Either they succeed or they don’t, either they find certain flaws or they don’t. But that is just a snapshot in time; after the PT is over, the pen-tester leaves, and the next day something could change for the worse.
<br>
But a BAS system is ever-present. It is always on the lookout, always awake and seeking attack paths to reach its goal. For example: if 6 months from now – someone accidentally changes a rule in the firewall to “Any – Any – Allow”, and with that – connects two disparate networks to each other, a BAS system would quickly learn of that and utilize it for further lateral movement.
<br>
I therefore like to think of a BAS product as a Terminator, but the one in the movie Terminator II; i.e.: he’s on our side. One of the good guys. ;)

## Don’t Tell Me I Have Problems
A typical PT by many cyber-security consulting firms ends with a report, which states the course of actions taken by the pen-tester, the attack paths they have tried (with the corresponding technical details), and what they have accomplished; each finding is rated with a severity, and usually – a “best-practices” textbook mitigation for the problem, which in many cases is banal or not suited for the organization.
As my boss likes to say: “Don’t tell me I have problems – I know that myself. What I need are reality-based effective solutions”.

A classic example of a “best practice” textbook advice appearing in a PT report is: _“Reduce the number of Domain Admin accounts”._
While it is advised to revise the need for each DA account from time to time – a security-aware organization may need all of its DA accounts it currently has, and cannot reduce their numbers.
<br>
A more reality-based mitigation for that organization would be to add those DA accounts to the “Protected Users” active directory group, thus forcing them to authenticate only using Kerberos (and not NTLM, or other even less secure protocols).
<br>
This demand for solutions (and not just shining light at problems) should be requested not only from human pen-testers, but also from AI-based ones. A good BAS product should not just write reports of attack paths, but also advise on solutions or mitigations for the problems found.

## Who should buy a BAS system?
This has to do with organizational maturity. There’s no need to buy an expensive BAS solution when other (perhaps more affordable) solutions can find low hanging fruit. While some BAS solutions can also find certain vulnerabilities due to unpatched systems – it may be better and more cost-effective to use a dedicated product for vulnerability scanning.
<br>
Same goes for certain legacy protocols and misconfigurations, such as the LLMNR and WPAD protocols, or Active Directory overly permissive ACLs. A BAS solution might find those and exploit them as part of an attack path – but good ol’ Windows hardening best-practices or a commercial product which specifies in seeking and alerting on such Active Directory “dark corners” might give us more bang for the buck.

## PT Is Dead, Long Live the BAS?
We are living at the frontier of the AI age, so what I am stating here might seem irrelevant (if not outright wrong) in the not-too-distant future, but ever since BAS products have become a thing – company CEOs (and the tech media outlets) have been wondering: is the human Pen-Tester a thing of that past? To be replaced by BAS systems?
<br><br>
The short answer, in my honest opinion is: **no!**
<br><br>
Manual pen-testing involves using a team of cybersecurity experts to manually examine systems and infrastructure for vulnerabilities, misconfigurations and logic flaws. This process can be time-consuming and expensive, but it provides a level of customization and realism that automated tools cannot match. Automated pen-testing, on the other hand, involves using software tools (AI and Machine Learning or not) to “scan” systems and infrastructure for the aforementioned problems, merely emulating a human attacker. This method is faster and more cost-effective than manual testing, but misses certain attack paths that an experienced human pen-tester would catch.
<br>
Are these two approaches mutually exclusive? – not at all. They can benefit from each other! A mature organization can use basic automated tools to find the low hanging fruit (unpatched systems, known misconfigurations), and challenge a BAS product to find more complex attack paths, in order to shine light on certain “dark corners” of the organization. Then, every designated time period, or when the need arises (such as a new system is scheduled to run in production) – the org can also request a manual PT, giving the human pen-tester a challenge to find high hanging fruit for a change.
<br>
They will even thank you for it!