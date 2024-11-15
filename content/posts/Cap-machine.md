---
title: Cap HTB CTF Machine
date: 2024-11-14T21:04:01Z
lastmod: 2024-11-14T21:04:01Z
author: Marcelo Bregieira
avatar: /author.jpg
# authorlink: https://author.site
cover: /cap.png
images:
   - /cap1.png
   - /cap2.png
   - /cap3.png
   - /cap4.png
   - /cap5_v2.png
   - /cap6.png
   - /cap7_v2.png
   - /cap8.png
   - /cap9.png
   - /cap10.png
   - /cap11.png
   - /cap12_v2.png
categories:
  - CTF
tags:
  - IDOR
  - Log Analysis
  - Web Application
  - Python
nolastmod: true
draft: false
---

Cap is an Linux machine running an HTTP server that performs administrative functions including performing network captures. 

<!--more-->

Let's Get to Work!

First of all, like with all other machines, we have an IP address. Let's start with enumeration.
<img src="/cap1.png" alt="nmap result" style="width:100%; max-width:600px;">
We see that three ports are open: 21 (FTP), 22 (SSH), and 80 (HTTP). 

Let’s take a look at the HTTP service.

We’re presented with a dashboard. As we explore the site, which doesn’t have many active features, we navigate to Security Snapshot.
<img src="/cap2.png" alt="nmap result" style="width:100%; max-width:600px;">
From the URL, we notice it’s set to http://10.10.10.245/data/245.
<img src="/cap3.png" alt="nmap result" style="width:100%; max-width:600px;">
Examining the requests through BurpSuite, we see there’s no cookie or ID associated with it.

**Smells like a vulnerability!**


We change the URL to another number (a lower one) and can see other users' data! Generally, the first entries tend to be the most interesting.
<img src="/cap4.png" alt="nmap result" style="width:100%; max-width:600px;">
So, we go to http://10.10.10.245/data/0 and download the file, which results in a .pcap file that we can analyze in Wireshark.

Analyzing the file, within the FTP protocol, we find a password in plain text, giving us the credentials nathan:B<**snip**>!.
<img src="/cap5_v2.png" alt="nmap result" style="width:100%; max-width:600px;">
Let’s move on to the other services, starting with FTP. Using the credentials obtained from the .pcap file, we gain access!
<img src="/cap6.png" alt="nmap result" style="width:100%; max-width:600px;">
From there, we retrieve our first flag in user.txt.
<img src="/cap7_v2.png" alt="nmap result" style="width:100%; max-width:600px;">
Now, let’s try the SSH service. Using the same credentials, we gain access again!
<img src="/cap8.png" alt="nmap result" style="width:100%; max-width:600px;">
Now, we want to escalate privileges to root! Using the command **getcap -r / 2>/dev/null**, we check which binaries have elevated privileges.
<img src="/cap9.png" alt="nmap result" style="width:100%; max-width:600px;">
We notice that python3.8 has some elevated capabilities. Let’s investigate further.
After a quick search on GTFObins, we find that if CAP_SETUID is enabled on the Python binary, we can escalate privileges.
<img src="/cap11.png" alt="nmap result" style="width:100%; max-width:600px;">
As we can see from our getcap output, it’s enabled. So, using the command suggested by GTFObins, we escalate our privileges to root.
<img src="/cap10.png" alt="nmap result" style="width:100%; max-width:600px;">
And that’s how we obtain the root flag in the root folder.
<img src="/cap12_v2.png" alt="nmap result" style="width:100%; max-width:600px;">
Another machine completed! On to the next one!

