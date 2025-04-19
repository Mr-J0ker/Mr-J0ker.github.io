---
layout: post 
title: Jerry 
date: 13 Apr 2025
---
# Summary
This is a Windows box rated as easy. 

Initial Enumeration shows only one possible way into the box - Apache Tomcat hosted onport 8080. Default credentials will provide a way into Tomcat where we can easily pop a reverse shell as SYSTEM by deploying a war file. 


# Initial Enumeration
Running the usual nmap scan, it appears there is only one possible path - port 8080.
![nmap](/docs/assets/images/htb/Jerry/nmap.png)

# Looking at Port 8080
This appears to be Tomcat default page. With only Tomcat and nothing else to go on, I will run Hydra to go through a set of username:password.  
![tomcat login](/docs/assets/images/htb/Jerry/tomcat login.png)
![hydra results](/docs/assets/images/htb/Jerry/hydra bruteforce.png)  
Not sure why Hydra returns two set of results but the user 'tomcat' is valid credential that works on this installation.

# Getting a reverse shell
With Tomcat, getting a reverse shell is relatively straightforward - just use msfvenom to create a malicious war file, upload and deploy it. 
![tomcat war upload](/docs/assets/images/htb/Jerry/tomcat war upload.png)
Tomcat will show us how to trigger the deployed war file.
![uploaded revshell](/docs/assets/images/htb/Jerry/uploaded revshell.png)

Triggering the war file gives us shell as SYSTEM. 










