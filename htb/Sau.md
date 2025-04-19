---
layout: post 
title: Sau 
date: 19 Apr 2025
---
# Summary
This is a Linux box rated as easy. 

Initial enumeration shows probably only one way into the box - via the application running on port 55555. Researching on it, it appears there is a SSRF vulnerability on it. SSRF is a vulnerability that allows us to reach network resources that we are not able to from outside the network. Port 80 appears to be the obvious candidate since it is shown as filtered in nmap scan results. We then reach MalTrail where another vulnerability will allow us RCE. Escalating privileges to root is trivial from there.

# Initial Enumeration
Running the usual nmap scan, it appears there is only one possible path - port 55555.
![nmap](/docs/assets/images/htb/Sau/nmap.png)

# Looking at Port 55555
Browsing the page we land on what appears to be request-baskets.  
![landing_page](/docs/assets/images/htb/Sau/landing_page.png)
Usually when I hit a site that is not custom-made, I will look around for vulnerabilities associated with it and it appears that version 1.2.1 is vulnerable to SSRF - the same version that is hosted as seen by the footer at the end of the page. Reading up on this vulnerability, it appears to be the forward_url parameter when creating a new basket that will result in all requests to our basket being routed to the destination indicated in forward_url. I don't usually see port 80 filtered when running scans on boxes, so this makes localhost port 80 an attractive target to try reaching. 

Using Burp Suite repeater, I added the necessary parameters - forward_url and proxy_response, to route all requests to my basket to localhost port 80.
![Routing the request](/docs/assets/images/htb/Sau/ssrf.png)

Visit our basket - we are now routed to port 80 which appears to be serving Maltrail.
![Maltrail](/docs/assets/images/htb/Sau/maltrail.png)

# RCE via Maltrail
Once again, we look up for vulnerabilities associated with Maltrail v0.53 - version information is available on the footer. There appears to be a Command Injection vulnerability on the login page that can be exploited via the username parameter. This is the exploit I used: [Spookier](https://github.com/spookier/Maltrail-v0.53-Exploit). 
![Running the exploit](/docs/assets/images/htb/Sau/exploit.png)
![Reverse shell](/docs/assets/images/htb/Sau/revshell.png)

# Privilege Escalation
With the reverse shell, a quick check shows that our user is able to run systemctl status without supplying any password. As per GTFObins, if we manage to enter the pager, we can run commands at elevated privileges. This then give us root access on the box. 
![Privilege Escalation](/docs/assets/images/htb/Sau/privesc.png)










