---
layout: post
title: 	"Red vs. Blue - Networking"
description: "The final boss of RvB"
tags: [swift, networking]
---

# What is RvB?
Red vs. Blue (RvB) is a SWIFT-run competition that is is modeled after CCDC. A team of 4 students is placed into a vulnerable network that they have to protect. SWIFT board, alumni, and volunteers act as an active red team, trying to compromise the network and lay out persistence on as many machines as possible. This gives students a unique experience of troubleshooting and incident response in an active breach scenario.

Preparation and development of an RvB-style competition involves few major steps: brainstorming the setting, creating vulnerable machines, configuring the networking, and setting up a scoring engine.

To read more about RvB, check out this blog by [CPP SWIFT](https://www.calpolyswift.org/blog/rvb-2022-spring).

# Goals of this post
This is not really meant to provide a techincal guidance or step-by-step on how to configure networking for an RvB-style setup. I will explain some techincal terms here and there, but this is mainly to show how the networking for RvB has changed over the years, discuss about the thought process behind each iteration, and provide additional insights to help the reader better understand the inner mechanisms of RvB networking.

# The previous RvB networking configuration

{% include image.html path="blog/rvbnetworking/RvB-Topology-previous.drawio.png" path-detail="blog/rvbnetworking/RvB-Topology-previous.drawio.png" alt="" %}

This was how RvB was previously setup. After the previous networking guy left, this was basically what I was left with in terms of documentation.

# Primer to ESXi networking terminology
We run RvB mostly off ESXi, which is a Type-1 hypervisor by VMware. ESXi has the ability to create virtual switches, or vSwitches. vSwitches operate pretty much exactly the same as physical switches.

vSwitches contain port groups. I like to think of port groups as "labels" for the virtual ports on the vSwitch.

If you create two port groups on a single vSwitch and connect VMs to both of those port groups, it would operate the same as connecting computers to different physical switchports on a single physical switch.

You can also configure VLANs and other networking metrics on individual port groups. I will touch more on this topic later in this blog post.

(Not to be mistakened with switching features like LACP or port bonding)

# 1-to-1 NAT
The first thing I focused on was reimplementing the 1-to-1 NAT between the pod networks and the core networks. Every system on each pod needed to be individually accessible from the core network (AKA have its own IP address in the core network subnet range).

After much trial and error, this was the NAT configuration that allowed us to achieve 1-to-1 NAT:
- Under System → Routing, ensure that the default gateway is configured to be the Core Router's LAN interface (172.16.0.1).
- Under Firewall → Virtual IPs, create a new entry with the following settings:
	- Type - Proxy ARP
	- Interface - WAN
	- Address Type - Network
	- Address(es) - 172.16.X.0/24 (X being the team number)
- Under Firewall → NAT, create a new 1:1 mapping:
	- Interface - WAN
	- Address Family - IPv4
	- External subnet IP → WAN address
	- Internal IP → LAN Net
	- Destination - Any

# VLANs 
VLANs, or virtual LANs, is a feature of switches that allow you to logically separate broadcast domains on a single switch.

Simply put, with a standard switch without VLANs, you can only attach one network onto that switch. If you want to have two networks, you would need two switches.

{% include image.html path="blog/rvbnetworking/RvB-Topology-no-vlan.drawio.png" path-detail="blog/rvbnetworking/RvB-Topology-no-vlan.drawio.png" alt="" %}

With VLANs, you can attach multiple networks on a single switch.

{% include image.html path="blog/rvbnetworking/RvB-Topology-vlan.drawio.png" path-detail="blog/rvbnetworking/RvB-Topology-vlan.drawio.png" alt="" %}

In this example, notice how we reduced the number of switches when we implemented VLANs. If I create new VLANs, I can add many more networks onto that one single switch! That's one of the benefits of VLANs - less infrastructure overhead when creating new networks.

Now with VLANs in mind, let's go back to RvB networking.

One of the issues with the current setup of RvB is having too many vSwitches – each pod would require its own vSwitch, which is kind of unnecessary since I would have to create an entirely new vSwitch with a new port group for each team. Our lab eventually became a mess of vSwitches and port groups, I wanted to reduce creating new vSwitches whenever possible. So I configured each RvB pod's port group to belong in a different VLAN. These VLANs only existed within the port groups - the VLANs were not configured on any physical switch or router. This would make it so that each pod is isolated into their own separate broadcast domain, but still be connected to one vSwitch. Therefore, we can still create pod networks without having to create additional vSwitches.

{% include image.html path="blog/rvbnetworking/RvB-Topology-topo-vlan.drawio.png" path-detail="blog/rvbnetworking/RvB-Topology-topo-vlan.drawio.png" alt="" %}

# Infrastructure Upgrades
We also had an upgrade to our lab - instead of having individuals ESXi machines, we created an ESXi cluster with vMotion and iSCSI configured to allow for load balancing and other cool VM stuff. However, instead of using vSwitches, we had to use distributed switches, or dSwitches for short. vSwitches and dSwitches do pretty much the same thing, except vSwitches are unique per ESXi, while dSwitches exist for all ESXi's in a cluster.

I like to draw dSwitches as a very wide switch.

{% include image.html path="blog/rvbnetworking/RvB-Topology-dswitch.drawio.png" path-detail="blog/rvbnetworking/RvB-Topology-dswitch.drawio.png" alt="" %}

When we configured dSwitches initially, we were prompted for many other options, like uplinks, which I wasn't really familiar with so I just skipped those steps. After creating our dSwitch and our port groups with unique VLAN IDs, we ran into another issue. The issue was that machines on different ESXi's weren't able to communicate with each other. After much troubleshooting, I figured out that the issue was because the dSwitches weren't configured with uplinks. Uplinks allowed the dSwitch to operate across multiple ESXis. Without the uplink, the dSwitch would still exist across the ESXis (you can still add port groups and connect VMs to those port groups) but each ESXi would treat the dSwitch as their own local vSwitch. This means that VMs on the same ESXi on the same dSwitch can communicate with each other, but VMs on different ESXis but on the same dSwitch can't communicate with each other.

The way that I was able to solve this issue was to create the uplinks. But how? This was how:

In my setup, the uplinks were attached to each ESXi's vmnic0, which was configured as a trunk port. Each of the vmnic0's were connected to switchports on some HP switch that were configured as trunk VLANs with VLANs 1000-2000 being allowed through the ports. These VLANs only existed on the switch level; no routers contained any configurations for VLANs 1000-2000.

This would allow a user to create a port group with a unique VLAN ID, and be able to have VMs on that port group communicate with each other regardless of which ESXi the VM belonged on. Things like creating VMs, vMotion, and load balancing cause VMs to be automatically placed and migrated in different ESXi's, making this configuration an ideal way to abstract networking on the individual ESXi level.

{% include image.html path="blog/rvbnetworking/RvB-Topology-topo-infra.drawio.png" path-detail="blog/rvbnetworking/RvB-Topology-topo-infra.drawio.png" alt="" %}

# Replacing the Core Router
After reviewing the topology, I noticed that having a separate pfSense VM as a core router is repetitive. The core router's purpose was to create a 172.16.0.0/16 network. We can just make a new VLAN on the physical router with the 172.16.0.0/16 network and have that VLAN also be routed through the ESXis.

Also, the red team machines and the scoring engine are now placed on the 172.16.0.0/16 network. There's really no reason to have a separate red team subnet. The less networks, the better.

# Final Topology

{% include image.html path="blog/rvbnetworking/RvB-Topology-topo-final.drawio.png" path-detail="blog/rvbnetworking/RvB-Topology-topo-final.drawio.png" alt="" %}

# Final Thoughts
When viewing the previous RvB setup, the networking was a lot more simpler and easy to understand. VLANs made sense as well. However, when vSphere clustering and dSwitches became involved, traditional networking convention was thrown out the window. Having to understand the function of dSwitch uplinks, implemeting VLANs into vmnics and dSwitches, and how to figure out how to bridge the connection between VMware networking and physical networking was the biggest hurdle for me.