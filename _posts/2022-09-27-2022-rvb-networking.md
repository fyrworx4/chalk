---
layout: post
title: 	"Red vs. Blue - Networking"
description: "The secrets of RvB networking uncovered"
tags: [swift, networking]
---

# What is RvB?
Red vs. Blue (RvB) is a SWIFT-run competition that is is modeled after CCDC. A team of 4 students is placed into a vulnerable network that they have to protect. SWIFT board, alumni, and volunteers act as an active red team, trying to compromise the network and lay out persistence on as many machines as possible. This gives students a unique experience of troubleshooting and incident response in an active breach scenario.

Preparation and development of an RvB-style competition involves few major steps: brainstorming the setting, creating vulnerable machines, configuring the networking, and setting up a scoring engine.

To read more about RvB, check out this blog by [CPP SWIFT](https://www.calpolyswift.org/blog/rvb-2022-spring)

# Goals of this post
This is not really meant to provide a techincal guidance or step-by-step on how to configure networking for an RvB-style setup. I will explain some techincal terms here and there, but this is mainly to show how the networking for RvB has changed over the years, discuss about the thought process behind each iteration, and provide additional insights to help the reader better understand the inner mechanisms of RvB networking.

# The previous RvB networking configuration

{% include image.html path="blog/rvbnetworking/RvB-Toppology-previous.drawio.png" path-detail="blog/rvbnetworking/RvB-Toppology-previous.drawio.png" alt="No. 1 in the world" %}