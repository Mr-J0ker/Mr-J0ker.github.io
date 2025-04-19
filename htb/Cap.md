---
layout: post 
title: Cap
date: 6 Apr 2025
---
# Summary
This is the first box in HTB Intro to Red Team track. It is a Linux box rated as easy. 

Initial enumeration show that the likely entry point seems to be the webapp. The webapp itself has limited features. It appears that a feature to download pcap files leaks the user credentials that allows me to capture user.txt and get a foothold on the machine. Privilege escalation is relatively straightforward after via Linux capabilities.  

# Initial Enumeration
Running the usual nmap scan, it appears there are limited services exposed - FTP, SSH, HTTP.
![nmap results 1](/docs/assets/images/htb/Cap/enum1.png)
![nmap results 2](/docs/assets/images/htb/Cap/enum2.png)
![nmap results 3](/docs/assets/images/htb/Cap/enum3.png)

FTP does not allow anonymous login, and at this point we have no SSH credentials. Rather than attempting to bruteforce, HTTP is likely to be what will offer me a way onto the box.

# Enumerating the web app
The application looks relatively straightforward.
![webapp screenshot](/docs/assets/images/htb/Cap/dashboard.png)
As usual, I got feroxbuster running to look for subdomains while I start to manually explore the site. It appears that all options under 'Nathan' do nothing except to bring us back to the same page, except for perhaps 'Log Out'.

Looking at the other menu options, it appears to map as below:
- Security Snapshot: /capture -> /data/{n}
- IP Config: /ip
- Network Status: /netstat

Looking at ip and netstat I cannot help but try if command injection was possible but of course this is not the vulnerability on this app. I tried the 'Search' function but it appears to be doing nothing as well, pretty much ruling out the possibility of any SQL/command injection there. 

Admittedly I got lucky with this. Seeing no other way I thought it has to be related to the data captured by the pcap file and decided to revisit the option. It is then that feroxbuster shows that under /data, there is /1 and /2 as well. 
![feroxbuster results](/docs/assets/images/htb/Cap/feroxbuster.png)

Thus, I decide to fire up Burp Suite as a proxy and enumerate for which are the valid endpoints. Setting the payload position and settings with Intruder, it appears that /data/0 may have some data whereas /data/1 seems to be a huge file which I guess is capturing my feroxbuster scan. 
![intruder payload](/docs/assets/images/htb/Cap/intruder_payload.png)
![intruder_position](/docs/assets/images/htb/Cap/intruder_position.png)
![intruder results](/docs/assets/images/htb/Cap/intruder_results.png)

# Examining 0.pcap
Downloading 0.pcap, I fired up wireshark and had a look at the conversations in the file. It appears that there is FTP traffic in this file.
![pcap conversations](/docs/assets/images/htb/Cap/wireshark_conversations.png)

And we get the credentials for Nathan after filtering for FTP. 
![pcap ftp](/docs/assets/images/htb/Cap/pcap_ftp.png)

# Accessing FTP for user.txt
Accessing FTP with Nathan's credential, we manage to retrieve user.txt.
![user.txt](/docs/assets/images/htb/Cap/ftp_user.txt.png)

# Privilege Escalation
Reusing Nathan's credential, we managed to get a foothold on the box via SSH. This time round I enumerated for privilege escalation manually without linpeas. The usual sudo, suid, crontab all returns nothing. However, it appears that python has capability set on it. This is documented in gtfo and can be used for privilege escalation. And with that, I get root access to the box.
![python cap](/docs/assets/images/htb/Cap/python_cap.png)
![root.txt](/docs/assets/images/htb/Cap/root.txt.png)

# Post-root exploration
With the box done, I thought I'll take a look at the application itself. It seems that I was right - there is no specific function to deal with the 'search' function in the app. The pcap feature do indeed rely on granting python the capability so it is able to use tcpdump to capture the pcap file. 









