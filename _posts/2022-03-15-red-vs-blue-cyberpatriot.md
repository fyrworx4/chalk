---
layout: post
title: 	"Red vs. Blue - Season 2"
description: "Expanding the Red vs. Blue saga."
tags: [cybersecurity, swift]
---

Ever since we held our Red vs. Blue last semester, we've made significant changes on the way Red vs. Blue was managed - both from a techincal standpoint and a logistical standpoint.

It was kind of like a dry-run for our high-school version of Red vs. Blue, where we train CyberPatriot teams for Nationals - the original intent of this event.

# Task Delegation

Last semester, I had to step in as the project manager for Red vs. Blue - making sure that boxes were created, networking configured, etc.

This semester, that changed. My eboard members took the charge of managing Red vs. Blue and my main role was listening to status updates. And network troubleshooting (I will get to that in a bit!)

# The Scoring Engine

The [old scoring engine][old-se] had many problems:
- Too complex.
- Service pollers are too slow.
- Not enough support for scored services.

So we looked into a [new scoring engine][new-se], which had a ton more features and looked nicer. We needed to do lots of troubleshooting to get this to work, but overwall it was easier to manage and displayed the scores in a better format.

# Network Troubleshooting

One of the previous issues we had during Red vs. Blue in the fall was that red team couldn't use their own Kali VMs on our host machines - they had to use VMs that were hosted in our lab. In an ideal world, red team would use their customized Kali VMs to VPN into the TelcoLab and be able to perform reverse shells from blue team machines.

But we promised this for next Red vs. Blue, so we had no choice but to deliver.

And the solution was relatively simple - we just needed to add static routes going to the VPN servers. That way, traffic destined to the VPN server's internal network can be routed through the TelcoLab.

At least we thought - red team still couldn't get reverse shells. So I looked into the issue for a few days by running tcpdump, using ping and traceroute, running `python3 -m http.server` on various hosts on the network and attempting to curl from various hosts on the network.

Ping worked, but any Layer 4 traffic didn't go through. It was really confusing, so I looked into this even more. What could possibly be blocking Layer 4 (and above layers) traffic?

It was UFW.

Specifically, the VPN servers were running UFW and the default rules were in place.
- Deny incoming traffic
- Allow outgoing traffic
- Deny routed traffic

There were some rules that allowed specific traffic, but the main culprit was that UFW was **denying routed traffic.**

So by seting UFW to allow routed traffic, red team can now get reverse shells.

# AWS Mishaps

For one of the Red vs. Blue competitions, we tried out AWS to host our entire infrastructure. We estimated that it'll be around $200 to host a 4-hour competition - if we use spot instances.

But for some reason, we didn't use spot instances - it make the automation too complicated so we resorted to using On-Demand instances, which increased our costs around 4x. Green Team also went *ham* with creating AMIs and restoring EC2 instances (On-Demand!), which generated a lot of EBS overhead. We also needed to spin up tons of Kali machines because they kept breaking.

It wasn't fun, and I am now $2.6K in debt.

# Red Teaming Fun

Besides destroying my bank account, I got to red team this semester! I feel like I'm becoming part of the dark side by learning more about pentesting. I think this really opened up my interests and potential career path into red teaming.

I just need to do more HTB.

See you on the other side!

[old-se]: https://github.com/fyrworx4/PulseEngine-ScoringEngine
[new-se]: https://github.com/scoringengine/scoringengine