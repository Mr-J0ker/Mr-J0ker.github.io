---
layout: post 
title: Knife 
date: 6 Nov 2025
---
# Summary
This is a Linux box rated as easy. 

Initial enumeration does not show many available openings. The web application returns an unusual header that turns out to be what gives the foothold on the machine. Privilege escalation is relatively straightforward after, though care needs to be taken since the exploit used is not an immediate reverse shell.

# Initial Enumeration
Running the usual nmap scan, it appears there are limited services exposed - SSH, HTTP.
![nmap results](/docs/assets/images/htb/Knife/nmap_results.png)

As usual, brute-forcing SSH immediately is not my kind of thing, so enumerating the web app looks to be the immediate path. 

# Enumerating the web app
The application is a single page - no other clickable links for further exploration.  
![webapp screenshot](/docs/assets/images/htb/Knife/mainpage.png)

Set Nikto and feroxbuster to run together. While feroxbuster does not seem to reveal any further directories, Nikto does reveal an unusual header response. 
![unusual_header_value](/docs/assets/images/htb/Knife/nikto.png)

Here we can see the x-powered-by header to be PHP/8.1.0-dev. A quick search on searchsploit shows 49933.py - a vulnerable in the user-agent where there is a backdoor we can use. 
![php_dev_backdoor](/docs/assets/images/htb/Knife/php_backdoor.png)


# Getting a foothold
With this, getting a foothold is trivial.   
![foothold](/docs/assets/images/htb/Knife/foothold.png)

# Privilege Escalation
Examining the exploit code we can see it is basically injecting command via the 'User-Agentt' header. 
![exploit_code](/docs/assets/images/htb/Knife/exploit_code.png)

We will need to have a better revshell to allow us to escalate our privileges on this box, because privilege escalation is straightforward as well with the user's permissions. 
![bash_revshell](/docs/assets/images/htb/Knife/bash_revshell.png)
![root](/docs/assets/images/htb/Knife/root.png)










