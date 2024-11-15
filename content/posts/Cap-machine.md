---
title: Cap HTB CTF Machine
date: 2024-11-14T21:04:01Z
lastmod: 2024-11-14T21:04:01Z
author: Marcelo Bregieira
avatar: author.jpg
# authorlink: https://author.site
cover: cap.png
# images:
#   - cap.png
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

<img src="cap.png" alt="Cap HTB CTF Machine" style="width:100%; max-width:600px;">

Improper controls result in Insecure Direct Object Reference giving access to another user's capture. The capture contains plaintext credentials and can be used to gain foothold. A Linux capability is then leveraged to escalate to root. 
