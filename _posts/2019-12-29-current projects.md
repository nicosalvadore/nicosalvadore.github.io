---
layout: post
title:  "My current projects at work & at home"
date:   2019-12-29 14:32:08 +0100
tags: networking systems
---
# Implementing FreeIPA at work
We have about 40 VMs, running for the majority CentOS 7, to host our voice application, web servers, billing services, security mechanisms, & more... But we are lacking a central authentication service, and thus are using many local user and root accounts for each VM. Which is definitely a pain to manage and could introduce security concerns. I'd like to implement an *Identity Management* server to centralize those accounts, implement MFA, and give more granular *nix permissions for service accounts.


FreeIPA is the upstream project for RedHat IdM, it seems to answer to all of our needs and has a very good documentation in Red Hat KB. That's why I've decided to go with this solution. The benefits of FreeIPA are also :
- Good API (CLI & REST) and web interface
- DNS management from GUI (based on bind)
- Certificate Authority
- Sudo policies
- High availability deployment with 2 or more active servers

I'll be deploying FreeIPA soon in my work infrastructure and will slowly migrate our VMs to this new domain. I will also change the domain name of our resources because they are now created under a pretty long domain name, and I'm too lazy to type it entirely each time. But will be able to import the current DNS zone into FreeIPA to make this change rather painless.


# CCNA certification
As soon as I finished studying in school and got my CFC (Federal VET Certificate) in computer science / IT / (or whatever you want to name it) and got my foot into an internship at EPFL (Swiss Federal Institutes of Technology in Lausanne) in a team that was implementing the Cisco Unified Communications solution stack, I became interested more and more into network engineering.  
I was then dragged into the Cisco certification cursus and passed my CCENT in may 2017. But never got the courage to finish the complete exam to obtain a CCNA R/S. As my CCENT is expiring soon in 2020 and because Cisco is modifying most of their certification path in February 2020, I found myself motivated to obtain the CCNA at last.

I'm not particulary a Cisco *fanboy*, but the certification path is, IMO, a really good objective to follow early in a career. To be able to discover new technologies, network designs, and protocols, which are valid for all network vendors and not only Cisco. And it is very easy to find videos explaining each exam topic, detailed blog articles and documentations that are describing the protocols and helping us get that certification.

I'm using these resources to study my CCNA :
- CBT Nuggets videos from Jeremy Cioara for ICND2, I already used CBT Nuggets for ICND1 (CCENT)
- Todd Lammle's book on ICND2
- GNS3 to lab exam topics
- Cisco's PacketTracer to simulate switches in my labs
- /r/networking subreddit to lurk and be aware of what's beyond CCNA


# What's next
I have my CCNA exam scheduled for January 9th 2020, and then will be out of the office until February. 
When I return I'll be trying to deploy FreeIPA after a testing phase, which will be perfect to write a blog post about.
See you soon !
