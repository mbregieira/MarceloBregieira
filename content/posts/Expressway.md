---
title: Expressway HTB CTF
date: 2026-03-11T22:04:01Z
lastmod: 2026-03-11T22:04:01Z
author: Marcelo Bregieira
avatar: /author.jpg
# authorlink: https://author.site
cover: /expressway/expressway_logo.png
images:
   - /expressway/expressway1.jpg
   - /expressway/expressway2.jpg
   - /expressway/expressway3.jpg
   - /expressway/expressway4.jpg
   - /expressway/expressway5.jpg
   - /expressway/expressway6.jpg
   - /expressway/expressway7.jpg
   - /expressway/expressway8.jpg
   - /expressway/expressway9.jpg
   - /expressway/expressway10.jpg
   - /expressway/expressway11.jpg
   - /expressway/expressway12.jpg
   - /expressway/expressway13.jpg
   - /expressway/expressway14.jpg
   - /expressway/expressway15.jpg
   - /expressway/expressway16.jpg
   - /expressway/expressway17.jpg
   - /expressway/expressway18.jpg
   - /expressway/expressway19.jpg
   - /expressway/expressway20.jpg
   - /expressway/expressway21.jpg
   - /expressway/expressway22.jpg
categories:
  - CTF
tags:
  - ike
  - sudo
nolastmod: true
draft: false
---

ExpressWay is an Linux machine that focuses on enumeration beyond standard TCP scanning and introduces the exploitation of VPN-related protocols.

<!--more-->

We start by enumerating the target machine.
<img src="/expressway/expressway1.jpg" alt="nmap command" style="width:100%; max-width:600px;">
The scan shows only one open port: 22, which corresponds to SSH. 
<img src="/expressway/expressway2.jpg" alt="nmap tcp result" style="width:100%; max-width:600px;">
From both the TTL value (63) and the SSH banner, we can infer that the target is a Linux machine running Debian.

However, having only port 22 open is unusual for Hack The Box CTFs. Brute-forcing SSH without any prior user enumeration is rarely the intended path. In most cases, the solution lies in deeper enumeration, so the next step is to scan UDP ports.

<img src="/expressway/expressway3.jpg" alt="nmap udp command" style="width:100%; max-width:600px;">

After waiting quite a long time, we obtain the results.

<img src="/expressway/expressway4.jpg" alt="nmap udp result" style="width:100%; max-width:600px;">

Two ports immediately stand out:

 - 69 – TFTP
 - 500 – ISAKMP

We begin with TFTP and check whether Nmap provides any enumeration scripts for it.

<img src="/expressway/expressway5.jpg" alt="find command" style="width:100%; max-width:600px;">

We find two relevant scripts. We will try tftp-enum.nse to see if it reveals anything useful.

<img src="/expressway/expressway6.jpg" alt="nmap script tftp" style="width:100%; max-width:600px;">

The script returns an interesting result: a file named ciscortr.cfg exists on the server. 

We will interact with the TFTP service and download it.

<img src="/expressway/expressway7.jpg" alt="tftp help" style="width:100%; max-width:600px;">
<img src="/expressway/expressway8.jpg" alt="tftp download" style="width:100%; max-width:600px;">

Now that we have the file, we can read its contents.

<img src="/expressway/expressway9.jpg" alt="ciscortr content" style="width:100%; max-width:600px;">

Inside the configuration file we find something interesting: a potential username ike.

However, performing a brute-force attack with only a username is generally inefficient, so further enumeration is required.

The next open port is ISAKMP, something I had never encountered before working on this box. 

So I took some time to study it.

ISAKMP stands for Internet Security Association and Key Management Protocol.
It is a protocol used to negotiate and establish Security Associations (SA) in IPsec.

Common ports associated with it are:

 - UDP 500 → IKE / ISAKMP

 - UDP 4500 → IPsec NAT-T (which is also open on this machine)

When a VPN client connects, it sends a packet to UDP port 500. The server responds and both sides negotiate parameters such as:

 - encryption algorithms

 - authentication methods

 - cryptographic keys

Once the negotiation is complete, a Security Association is established and the IPsec traffic becomes encrypted.

One important implication is that this process can be vulnerable to attacks such as IKE Aggressive Mode PSK cracking.

While researching this, I discovered the tool ike-scan (https://github.com/royhills/ike-scan), which can be used to interact with IKE services.

<img src="/expressway/expressway10.jpg" alt="ike-scan" style="width:100%; max-width:600px;">

Let’s try it.

<img src="/expressway/expressway11.jpg" alt="ike hash" style="width:100%; max-width:600px;">

From the returned value we can see that the handshake belongs to the user **ike**, and we obtain a hash that can potentially be cracked.

Time to crack it.

**This smells like password reuse!**

<img src="/expressway/expressway12.jpg" alt="hashcat identify" style="width:100%; max-width:600px;">

We can see that Hashcat correctly identifies the hash type.

<img src="/expressway/expressway13.jpg" alt="hashcat command" style="width:100%; max-width:600px;">
<img src="/expressway/expressway14.jpg" alt="hashcat result" style="width:100%; max-width:600px;">

Eventually, we successfully recover the password for the ike user.

Next, we test whether these credentials also work for SSH.

<img src="/expressway/expressway15.jpg" alt="ssh login" style="width:100%; max-width:600px;">

They do.

We can now log in and retrieve our first flag: user.txt.

One of the first things I like to do after gaining a shell on a Linux machine is check the sudo privileges.

<img src="/expressway/expressway16.jpg" alt="sudo privileges" style="width:100%; max-width:600px;">

Unfortunately, the user ike does not have any sudo permissions.

At this point, I turn to the good old LinPEAS.

<img src="/expressway/expressway17.jpg" alt="upload linpeas" style="width:100%; max-width:600px;">

We upload LinPEAS to the target machine and run it, paying close attention to anything highlighted in red, as these usually indicate potential privilege-escalation vectors.

<img src="/expressway/expressway18.jpg" alt="sudo version red" style="width:100%; max-width:600px;">

The first item highlighted in red is the sudo version: 1.9.17. Let’s investigate it.

<img src="/expressway/expressway19.jpg" alt="google sudo version" style="width:100%; max-width:600px;">

A quick search reveals CVE-2025-32463, a privilege escalation vulnerability affecting this version, exactly what we need.

Let’s see if the exploit works.

<img src="/expressway/expressway20.jpg" alt="download POC" style="width:100%; max-width:600px;">

We download the proof-of-concept (PoC).

<img src="/expressway/expressway21.jpg" alt="transfer POC" style="width:100%; max-width:600px;">

After transferring it to the target machine, we give it execution permissions and run it.

<img src="/expressway/expressway22.jpg" alt="POC works" style="width:100%; max-width:600px;">

We now have root access and root flag.

Another machine solved. On to the next one.


