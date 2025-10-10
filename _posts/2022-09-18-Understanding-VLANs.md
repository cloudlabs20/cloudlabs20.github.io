---
layout: post
title: Understanding VLAN tags and Trunks Ports
categories: [ Networking ]
tags: [VLAN, Switch, Trunk ports]
author: 
enable_toc: true
render_with_liquid: true
date: 2022-09-18 00:00:00 +0530
---

![figure1]({{site.url}}/assets/img/VLAN-application-diagram-image.jpg)

## What is a VLAN?

Imagine you're at a huge party. There are different groups of people chatting, but they're not all in one big noisy crowd. Instead, they’re grouped into smaller circles—friends, family, co-workers. Each group is having its own conversation, and they only talk to people within their circle.

A **VLAN** is like one of those smaller circles at the party. It allows you to divide a big network into smaller, more organized groups. Even though all the computers, phones, and devices are connected to the same physical network, they only communicate with devices inside their own VLAN (their own circle).

### Why Do We Need VLANs?

Imagine you're working in a large company with hundreds of computers. The accounting department doesn’t need to be in the same network group as the marketing team, and the guest Wi-Fi shouldn’t be able to access your internal servers. VLANs help **segment** these groups, so they operate as if they’re on totally different networks, even though they’re all using the same cables and switches.

In short, VLANs are used to:

- **Organize the network**: Keep devices grouped logically, like by department or function.
- **Improve security**: Prevent devices in one VLAN from accessing resources in another VLAN.
- **Reduce network congestion**: By limiting communication to smaller groups, you prevent unnecessary traffic.

---

## How Do VLANs Work?

VLANs separate network traffic at **Layer 2** of the OSI model. Layer 2 is all about **MAC addresses** and **broadcasting**.

### Communication Within a VLAN

Let’s say you have two computers, **Computer A** and **Computer B**, both in VLAN 10 (the marketing VLAN). These computers can communicate with each other without any issues. They can share files, send messages, and more, as if they’re on the same local network. The data they exchange is handled by the switches, which tag the data with VLAN information to make sure it only reaches devices in the same VLAN.

### Communication Between VLANs

But what if **Computer A** wants to talk to **Computer C** in VLAN 20 (the accounting VLAN)? Well, that’s not going to happen without some extra help. VLANs are designed to **isolate** groups, so devices in one VLAN can’t directly communicate with devices in another VLAN. To allow this communication, you need a **router** (or Layer 3 switch) to route the traffic between the VLANs.

---

## VLAN Tags: How Switches Know Who Belongs Where

When data is sent across a network, it’s sent in something called a **frame**. For VLANs to work, switches need to know which VLAN a particular frame belongs to. This is done using a **VLAN tag**.

### 802.1Q: The VLAN Tagging Standard

Here’s where **802.1Q** comes into play, it’s the industry standard for VLAN tagging. When a frame is sent from a device, the switch adds a little bit of extra information (the VLAN tag) to the frame. This tag tells other switches, “Hey, this frame belongs to VLAN 10!” That way, only devices in the matching VLAN can receive it. When the frame reaches its destination, the switch strips off the tag and delivers the data to the right device.
---

## Voice VLANs: Optimizing Networks for VoIP Phones

If you’ve ever seen a **VoIP phone** (Voice-over-IP phone), you know that many phones have an **extra network port**. This allows you to connect your computer to the phone, and then connect the phone to the network, all using just one physical cable. This setup is super convenient because it reuses the same cable for both the phone and the computer, but how does the network know which traffic is for the phone and which is for the computer?

This is where **Voice VLANs** come in!

### What is a Voice VLAN?

A **Voice VLAN** is a dedicated VLAN that is specifically used for voice traffic (like phone calls). It's designed to ensure that VoIP traffic is separated from regular data traffic. This separation helps improve the quality of your phone calls and keeps your voice data from getting slowed down by other network traffic (like big file downloads or video streaming).

In a typical setup:

- The **phone** is assigned to a **Voice VLAN**.
- The **computer** is assigned to a **data VLAN** (which could be the regular office network).

### How VoIP Phones Use VLANs

Let’s say your office has VLAN 10 for computers and VLAN 100 for VoIP phones. Here's how a VoIP phone with a **built-in switch** works:

1. The VoIP phone is plugged into the wall with a single network cable.
2. The phone assigns **voice traffic** (like phone calls) to **VLAN 100** (the Voice VLAN).
3. The phone also acts as a mini-switch and passes **computer traffic** to **VLAN 10** (the data VLAN).
   
This setup allows you to reuse the same network cable for both the phone and the computer, while keeping the voice and data traffic separated into their own VLANs. This is really handy when you want to minimize the number of cables but still keep the network organized and efficient.

### Benefits of Using Voice VLANs

1. **Improved Call Quality**: By separating voice traffic from regular data, you avoid congestion on the network, ensuring that phone calls are crisp and clear, without delays or interruptions.
  
2. **Prioritization of Voice Traffic**: Most networks give priority to traffic on the Voice VLAN. This means the network will make sure phone traffic is delivered faster, even if there’s a lot of other data being sent at the same time. This is often achieved using **Quality of Service (QoS)** settings, which prioritize voice over data.

3. **Simplified Cabling**: Instead of running separate cables for the phone and computer, you can use one physical cable, reducing clutter and costs.

4. **Easier Management**: If you ever need to troubleshoot or manage the network, having voice and data on separate VLANs makes it easier to identify where issues might be coming from.

---

## Tagged vs. Untagged VLANs: What’s the Difference?

Now let’s break down the difference between **tagged** and **untagged** VLANs—an important concept when dealing with switches and VLANs.

### **Untagged VLANs** (The Default Highway)

An **untagged VLAN** is like the default road that traffic takes. Devices connected to an untagged VLAN don’t need to worry about VLAN tags. This is because the switch **removes** the VLAN tag before delivering the data to the device. The device has no idea it's part of a VLAN, it just sees a regular connection.

Think of untagged VLANs as the **default network** a device is on when it connects. Each port on a switch can be assigned an untagged VLAN, and devices using these ports will communicate on that VLAN **without** seeing or using VLAN tags.

### **Tagged VLANs** (Carrying Multiple Passengers)

A **tagged VLAN** is different. Here, the switch **keeps** the VLAN tag as the data moves through the network. This is necessary when a single network cable (or port) is carrying traffic for **multiple VLANs**, like on a **trunk port**.

With tagged VLANs, the data is labeled with a VLAN tag so that each device on the other side knows which VLAN it belongs to. You can think of this like a car carrying people from different groups, each passenger has a name tag so the destination knows who belongs to which group.

### PVID (Port VLAN ID)

The **Port VLAN ID (PVID)** is the VLAN ID assigned to untagged traffic on a particular port. This means if a device connects to a port and doesn’t have any VLAN tags, the switch will assign the traffic to the VLAN corresponding to the PVID of that port.

Think of the PVID as a **default assignment**. If data comes in without any tags (untagged traffic), the switch will use the PVID to decide which VLAN that traffic belongs to. So, if a port has a PVID of 10, all untagged traffic will be sent to **VLAN 10**.

---

### Automatic vs. Manual PVID Adjustment

Some switch manufacturers will automatically adjust the PVID based on the **untagged VLAN** that you set for the switch port. This means if you configure a port to be untagged for VLAN 20, the switch will automatically set the PVID to 20 as well, so you don’t have to manually configure the PVID for that port.

However, for other switch vendors, you will need to **manually adjust** the PVID to match the untagged VLAN on that port. If this isn’t done, untagged traffic might not go to the correct VLAN, causing communication issues.

### PVID for Ingress Traffic

**Ingress traffic**: When data comes into a switch port (from a computer, for example), if it’s untagged, the PVID will determine which VLAN that traffic belongs to. In most cases, the ingress and egress VLAN will be where the same—untagged traffic will come in and go out using the same VLAN. However, some advanced network setups might use **different ingress and egress VLANs** for security or traffic routing purposes.

---

### Trunks vs LAGs

If you have multiple VLANs on your network, you need a way to move traffic between switches without losing the VLAN information. This is where **trunk ports** come in. A trunk port is a special port that carries traffic for multiple VLANs between switches. Think of it like a superhighway that all the VLANs can use to get from one switch to another, without mixing up their traffic.

![figure2]({{site.url}}/assets/img/VLAN_trunk_port.png)
_Source: https://www.adroitacademy.com/web-portal/img/blog/43604421.png_

Now, here's an important note: **VLAN trunks** are not the same as **LAG (Link Aggregation Group) trunks**. Sometimes, vendors (especially network equipment manufacturers) use the word **trunk** when referring to **LAG** connections, and this can be a little confusing.

So what’s the difference?

- **VLAN Trunks**: These are used to carry traffic for **multiple VLANs** between switches. They help VLANs on one switch communicate with VLANs on another switch by tagging the frames with VLAN IDs.
  
- **LAG (Link Aggregation Group)**: LAG, on the other hand, combines multiple physical network links into one logical link. This is done to **increase bandwidth** and provide redundancy. For example, instead of using one cable between two switches, you might use four cables as part of a LAG. This gives you four times the bandwidth and makes sure traffic can still flow even if one cable fails.

While both LAGs and VLAN trunks help optimize the network, **LAGs** deal with physical link redundancy and speed, while **VLAN trunks** manage traffic for multiple VLANs. It's like the difference between having multiple roads to the same destination (LAG) and marking those roads with signs so people in different groups (VLANs) know which one to take.

---

## A Practical Example: VLANs in Action

Let’s imagine you’re setting up a network for a school:

- You create **VLAN 10** for teachers.
- You create **VLAN 20** for students.
- You create **VLAN 30** for the guest Wi-Fi network.

Now, teachers can access sensitive school documents and share files within VLAN 10, but students in VLAN 20 won’t be able to see or access anything in the teachers’ VLAN. Similarly, guests connected to the Wi-Fi are isolated in VLAN 30, preventing them from getting into the main school network.

VLANs give you **control**, **security**, and **flexibility** all in one!

---

## ARP, VLANs, and Security

Remember how we talked about the **ARP table** earlier? It keeps track of IP addresses and their corresponding MAC addresses. Even with VLANs, devices will use ARP to figure out who’s on the network. But here’s a security tip: if a malicious device joins the wrong VLAN, it might still be able to learn some information using ARP requests.

That’s why many networks also use **ACLs** (Access Control Lists) or firewalls to provide extra security, preventing unauthorized devices from doing anything sneaky, even if they manage to connect to the network.

---

## Wrapping Up VLANs

So, to summarize:

- **VLANs** are like creating separate groups (or party circles) within a larger network. Devices in a VLAN can only communicate with others in the same VLAN unless special routing is used.
- **Voice VLANs** are used to separate VoIP traffic from regular data traffic, improving call quality and network performance, especially in setups where a VoIP phone and computer share the same network cable.
- **Tagged vs. Untagged VLANs**: Untagged VLANs are the default highway for traffic, while tagged VLANs are for when a port carries traffic from multiple VLANs.
- **PVID**: The Port VLAN ID (PVID) is what untagged traffic is assigned to when it enters the network. It’s the default VLAN for untagged traffic on a port. Some switches automatically adjust the PVID based on the untagged VLAN setting, while others require manual configuration.
- **VLAN trunks** carry traffic from multiple VLANs between switches, while **LAGs** combine multiple physical links to increase bandwidth and provide redundancy.
  
VLANs are a powerful tool for network organization, security, and performance. Whether you’re splitting up a school network, isolating sensitive company departments, or separating guest Wi-Fi from the main office network, VLANs keep the network organized and secure.


##### Sources

- <https://www.practicalnetworking.net/stand-alone/vlans/>
- <https://www.extnoc.com/learn/networking/what-is-virtual-lan>
