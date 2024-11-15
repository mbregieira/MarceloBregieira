---
title: Cap HTB CTF Machine
date: 2024-11-14T21:04:01Z
lastmod: 2024-11-14T21:04:01Z
author: Marcelo Bregieira
avatar: /author.jpg
# authorlink: https://author.site
cover: /cap.png
images:
   - /1123.png
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

Improper controls result in Insecure Direct Object Reference giving access to another user's capture. The capture contains plaintext credentials and can be used to gain foothold. A Linux capability is then leveraged to escalate to root.

<img src="/1123.png" alt="nmap result" style="width:100%; max-width:600px;">
