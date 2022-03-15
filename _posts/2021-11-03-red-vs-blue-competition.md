---
layout: post
title: 	"Red vs. Blue"
description: "The begnning of a new era for SWIFT."
tags: [cybersecurity, swift]
---
Red vs. Blue is an adversarial simulation competition to provide students with a unique, hands-on experience of defending Linux and Windows machines from an active threat actor. It's modeled after the CCDC competition to allow students to get the feel and rigor of a cyber competition, integrated with business objectives.

And it's free for everyone. No experience required.

This has been one of the most ambitious projects that our student organization, SWIFT, has ever taken on. We had around 40 members participate in our Red vs. Blue event, we brought in alumni from way before our time to help out with the red team, we've had both our CCDC and CPTC teams to assist with this project.

This was a large-scale project that involved an immense amount of collaboration since summer. Lots of concentration, troubleshooting, and all-nighters to get to this level of quality of a workshop - for students, by students. We're breaking new grounds.

And here's how we did it.

## The Setup

Red vs. Blue was originally a training exercise/scrimmage for high school CyberPatriot teams that were going to the Nationals-level competition, but last year our organization decided to make it available for college students as well. Preparing for this version of Red vs. Blue began in the summer, when my good friend Alex pitched a Star Wars themed idea.

### Pod-and-Core Network

Each team needed to have the same exact environment, so we decided to call these environments "pods". Each pod was connected to a pfSense router (called pod routers), and the pfSense routers were connected together in the "core" network by another pfSense router (called the core router).

All environment boxes on all pods need to be accessible with individual IP addresses, so 1:1 NAT was configured on each of the pod routers.

A red team network was added onto the core router, which Kali VMs were connected to, as well as the scoring engine.

### Vulnerable Machines

SWIFT's Director of Academy, Gabe, led the creation of vulnerable and misconfigured virtual machines. We had four machines in total:

- Windows Server 2012 (Communications)
- Windows 7 (Docking Bay)
- Ubuntu 20.04 (???)
- CentOS 7 (Engines)

Each of the machines have their own services that needed to be up to score points. In Red vs. Blue, the environment is supposed to align with the lore for realism, but having a Star Wars ship running Windows Server 2012 was one of the funniest things for me.

### Scoring Engine

The scoring engine, built from Python, scores service uptime by simulting users accessing business-critical services, such as using SSH to remotely log into an SSH server or initiating a RDP authentication request to an RDP server. Here's the [repo](https://github.com/fyrworx4/PulseEngine-ScoringEngine).

There were two components to the scoring engine - the user web interface and the pollers. Credits to Nathan and Silas for building/developing the user web interface (which you can access by going [here](http://scoring.oneoneone.one)). I worked with Gabe and a CCDC team member, Jacob, to add more capabilities to the scoring engine, such as scoring a MySQL and IRC server.

## The Competition

Setting up the environment was a project of its own, but the logitstics of how the event is run needs to be planned. We needed to answer questions such as:

- How are teams created? How will teams communicate?
- How difficult should the competition be?
- How do we send out announcements?

We settled on Discord to manage everything, both the internal operations of Red vs. Blue and the communication for teams. Alex set up an Apache Guacamole instance as a jump box where competitiors can access their team environment, so we needed to explain the process of how to connect to the environment using Apache Guacamole, especially to those who have never heard of a jump box before.

## Lessons Learned

Red vs. Blue was a very large project that took many hours to accomplish. But many of those hours were from the last two weeks before the event. I wished that we started earlier, because towards the competition, I had to start pinging everyone on Discord to update the team on the statuses on the boxes.

Personally, it gave me some insight on how to manage and delegate tasks. Working on Red vs. Blue has taught me a lot about patience and how to make sure that everyone's on the same page.

Also, we need to be more clear on what the objectives are. We tend to overestimate the skill level of our members, so we need to remind ourselves that simple tasks such as disabling SMBv1 is not as clear and cut as we might think.

But this is a unique opportunity that nobody else receives outside of CCDC/CPTC. A great start to Red vs. Blue, but we can do better.