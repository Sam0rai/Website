---
layout: striped-default
title: "Communication Obstacles On the Path to Problem Solving"
date: 2023-03-17
---
In our day to day job in IT (but actually, in all aspects of life), we encounter problems which need solutions, or have needs that need to be addressed and fulfilled.
In an ideal situation: people will have a clear understanding of their needs, or a comprehensive view of the problem; which in turn enables them to “break them down” to modular pieces, analyze them and come up with the optimal solution (given all necessary parameters and conditions), either alone or after brainstorming with others.
<br>
However, a lot more often than not – I have seen a lack of this type of analytical thinking; it starts with a lack of definitions for terms used, as well as laconic explanations of the needs or the problems that emerged, continues with mixing the problem with the solution, and ends up with explaining why the solution is in fact the real problem – and why the only real solution is to do X and not Y, or why a solution to the problem is not feasible in the first place!
<br>
The following paragraphs address 4 such common obstacles in communication skills regarding problems and the ways to solve them: 
<!--more-->
<br><br>

### 1. Undefined or Ambiguous Terminology
One of the most basic misunderstandings in interpersonal communication occurs when using terms which mean different things to different people. I had a lecturer in college (in a course about logic, nonetheless!), who kept referring to the subject of his sentences as “this”, while writing proofs on the board; by the fourth “this” into the proof –  75% of the classroom misinterpreted what the “this” written on the board was referring to.
<br>
I cannot count the number of meetings I’ve participated in, where uses-cases were discussed using wrong or ambiguous terminology, while trying to express the need or explain the problem at hand. A term which was brought up at the beginning of the meeting changed its definition over time, or the same term was used to define different things. This creates what I refer to as the “Tower of Babylon” effect, where people are communicating with each other, but misunderstanding one another, sometimes without even realizing it.
<br><br>
As one of my math teachers in college once put it:
> _“One must always start with the definition of terms. If you decide to skip that phase and merely go with the flow – then I don’t know where you’re gonna end up, but I shall waive to you Bon Voyage on your ill-advised sailings”_

For example: if you’re talking about a network architecture, using generic terms such as “host” or “proxy” will only confuse others and let them interpret those terms as they see fit; which “host” are you referring to? The endpoint or the server? What do you mean by “proxy”? The perimeter proxy at the organization’s gateway, or a proxy server which is part of the product you’re currently discussing?
<br><br>

### 2. Laconic Explanations
Maybe it’s the fault of technology, where all messages are short (Twitter), short-lived (Vine, TikTok) and summarized (ChatGPT); maybe not. But people have a tendency to be impatient and not elaborate their ideas, even when discussing complex structures, workflows or problems. This shallows the conversation, upsets the participants and when done via email – creates a lot of ping-pong communication.
<br>
One of my favorite questions to ask is: “What’s the story?”. Treat me as if I am a new member on staff, who has no prior organizational knowledge of the problem \ the product \ the service etc., and tell me the whole story; elaborate what the system does, what have you done in the project so far and where the problem lies.
<br>
This may sound like a lot to relay, but even a few paragraphs of explanation can suffice to bring someone up to speed, before dropping the problem on their lap and expecting an optimal solution from them.
<br><br>

### 3. Misdefining the problem
Misdefining the problem is a common mistake that many people make when trying to solve a problem; a misunderstanding of the problem and what they believe is the source of the problem. This can lead to wasted time and resources, as well as ineffective solutions.
<br>
For example: Let’s say we have a service that fails to connect to a remote web server via its API. One of the developers complains to IT that “the firewall is blocking outbound traffic to the web server, so you need to define a designated firewall rule to allow it!”. <br>
But that is not the problem; the problem is that API connections to the web server fail. While it could be the result of the firewall blocking the traffic, it could also happen due to various other reasons.
<br>
But since the developer inserted the solution into the definition of the problem, the firewall admins do as they are told, and create a new firewall rule, without even checking to see if there was any blocked traffic to begin with.
The new firewall rule doesn’t get any hits, and the admins tell the developer they don’t see any traffic going out; when in fact – there is outbound traffic to the remote server, but through another (more complacent) rule. After further investigation, it turns out that the traffic was dropped by the remote server itself due to a misconfigured iptables rule on that server. <br>
The bottom line here is: don’t include your solution in the phrasing of your problem or your needs.
<br><br>

### 4. Matching the wrong solution to the problem
I recall a meeting I was asked to participate in, which addressed the possible usages of Blockchain technology, the experiments that were already commencing using it, and the current problems and obstacles the designated IT team were facing. <br>
I have no expert knowledge in this field, so I just sat in my chair and listened to everyone, trying to learn and absorb what I could; certain thoughts ran through my mind, but I was a bit bashful to utter them. As the meeting came to a close, my boss (the CISO; also present at that meeting) looked at me and proclaimed that it looks as if I have something to contribute to the discussion; I don’t think he threw me under the proverbial bus, as much as gave me a nudge to jump into the water. All eyes on me now, huh?
<br><br>
“You’ve brought up some very interesting points here”, I nodded to no one in particular in the room. “But you’ve reminded me of the old Chinese proverb, that if a frog had wings – it would still bump its ass when it hopped”.
Silence filled the room; everyone looked a bit baffled. I, on the other hand, felt proud of misquoting a scene from the movie [Wayne’s World](https://clip.cafe/waynes-world-1992/where-did-learn-english/), creating my own proverb spin on it. Ok, time to explain the moral of the idiom to the crowd:
<br><br>
<div class="text-with-image">
  <p>
    <I>“You’ve been experimenting with Blockchain technology; developing smart contracts, uploading data or reading from the Blockchain… but you’ve been using it as a solution to a problem you’ve postulated was in fact the solution for to begin with; and once that postulation became an axiom, you kept on trying to find solutions to the problem your solution was creating for you.
    You are like a frog who somehow got wings put on their back. But instead of trying to fly with them in order to get around, you’re still trying to hop on the ground, and by doing so – you only encounter more problems, the wings interfere, slow you down and end up hitting your butts”.</I>
    <br><br>
    <b>Mic drop.</b>
  </p>
  <img src="/assets-striped/posts/2023-03-17-Communication-Obstacles-On-the-Path-to-Problem-Solving/Tree-Frog-With-Wings.png" alt="Tree Frog With Wings">
</div>

### To Conclude
When the problem is not fully understood or not communicated with greater detail to others, our chances to figure out a solution for it diminish, especially when working on it with others. Therefore, we must strive to:

- **Define the terminology** <br>
Make sure everyone is on the page regarding terms we’re using in the conversation (even if it seems obvious to us).
- **Elaborate** <br>
Explain all the necessary details, so your associates get a fuller understanding to the question: _“What’s the story?”_.
- **Analyze the problem** <br>
Investigate the actual source of the problem, and refrain from inserting your (biased) solution into the definition of the problem when explaining it.
- **Double-Check the solution** <br>
Is the solution in fact a solution, or is it causing us more problems down the road? <br>

Happy problem-solving!