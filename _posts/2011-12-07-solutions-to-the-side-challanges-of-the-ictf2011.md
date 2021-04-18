---
layout: post
title: Solutions to the side challanges of the iCTF2011
date: 2011-12-07
tags: ["ICTF"]
---

Solutions of our Team for the [iCTF 2011](http://ictf.cs.ucsb.edu/) [Challenges](http://challenges.ictf2011.info/).
<!--more-->

* * *
Challenge 05: (75 pts)

(required files: [9x9.pdf](http://www.ictf2011.info/files/9x9.pdf))

**Solution: "Valkyries**"

* * *
Challenge 09: (350 pts)

_Here at iCTF HQ, we have a little ADD problem.         Seeing how cheap domain were when we registered ictf2011.info, we decided to buy another domain.         There was a bulk discount!         Cool, ha?_

_Except, we forgot what the domain was.         Can you find it?_

_SQUIRREL!_

0x69637466(.info)
https://sites.google.com/a/0x69637466.info/ictf/
**Solution: "I@mD@Sh3rl0k0fth31nt3rn3tz**"

* * *
Challenge 10: (10 pts)

_Who would never laugh at the mules?_
(required file: [ictf2011.m4v](http://www.ictf2011.info/files/ictf2011.m4v))

At about 1:19 there are some frames exchanged.

**Solution: "Mule Herders"**

* * *
Challenge 11: (350 pts)

_A gpg secret key (ascii encoded) have been splitted into two parts and  concatenate into the lookatme file. The first part of the key is 872  bytes long and the second is 873 bytes long. Find the key and decrypt  the file secret.txt.gpg. Don't forget to include the key you found in  this following block before trying to decrypt the file. -----BEGIN PGP PRIVATE KEY BLOCK----- Version: GnuPG v1.4.10 (GNU/Linux)  KEY -----END PGP PRIVATE KEY BLOCK-----_

(required file: [findthekey.tar.gz](http://www.ictf2011.info/files/findthekey.tar.gz))

The key starts with "lQ", brute forcing a little bit from position 484764 to 484795 helped to find the key.

**Solution: FIRSTRULEOFBUSINESSPROTECTYOURINVESTMENT**

* * *
Challenge 13: (250 pts)

_When I was playing around with the backdoor I deployed on Zeus' laptop,         I found that it was very interested in this page.         Discover why, and if it's worth something, you'll get a good cut._

(required file: [inferno.html](http://www.ictf2011.info/files/inferno.html))

brainfuck code in html source, resolves to:

_Yo, Ben, here's your cut for the Zimmermann job in Quantico.  I know only you
can decode messages sent from the circles of hell, so don't try to complain
again that you never got the money.
Godspeed,
Enigma

(CB;:9]~}5Yz2Vw/StQr*)M:,+*)('&%$#"!~}'{zyx875t"2~p0nm,+jch'`%_

The last line is a malbolge script, which gives the solution.

**Solution: EvIl!**

* * *
Challenge 14: (50 pts)

_I read it encoded. Can you?_

(required file: [IReadItEncoded](http://www.ictf2011.info/files/IReadItEncoded))

The link leads to text which is base64 encoded. Decoding produces an ASCII QR-Code tag, which leads to the solution.

**Solution: Xis4n00bs**

* * *
Challenge 15: (150 pts)

_Why?_ (link to http://10.13.3.131:8080/)

**Solution: "5r3kc4h"** (Hackers backwards in leet)

* * *
Challenge 17: (100 pts)

_**l33786**_

_People! The new amazing l33786 processor is here! More than ten  powerful opcodes! Almost eight bits to the byte! Non-Uniform Memory  Access for super-high performance!_

_We are also super-green! To conserve power, memory in our system  is split in two halfs: low and high. Memory locations are physically  7-bit long, but this is not a limit for our CPU! Just store values below  127 in the low memory and values 128 or higher in high memory. There  are 4 high memory locations and 4 low ones._

_Two registers are provided, **<tt>rl</tt>** and **<tt>rh</tt>**. Low memory is wired to <tt>rl</tt>, high memory is wired to <tt>rh</tt>. Registers are not limited in the values they can store._

_With unprecedented generosity, UCSB is giving you access to a  version of l33786 that integrates an arithmetic coprocessor! The  coprocessor can increment or decrement arbitrary memory locations. DMA!  Amazing._

_Here is our state-of-the-art RISC instruction set:_

_<tt>ldh {addr}</tt> will perform:<tt>rh = high_memory[addr]</tt>_
_ <tt>ldh {addr}</tt> will perform:<tt>rl = low_memory[addr]</tt>_
_ <tt>sth {addr}</tt> will perform:<tt>high_memory[addr] = rh</tt>_
_ <tt>stl {addr}</tt> will perform:<tt>low_memory[addr] = rl</tt>_
_ <tt>incl {addr}</tt> will perform:<tt>low_memory[addr]++</tt>_
_ <tt>decl {addr}</tt> will perform:<tt>low_memory[addr]--</tt>_
_ <tt>inch {addr}</tt> will perform:<tt>high_memory[addr]++</tt>_
_ <tt>dech {addr}</tt> will perform:<tt>high_memory[addr]--</tt>_
_ <tt>xor {dest} {src}</tt> will perform:<tt>dest = dest XOR src</tt>_
_ <tt>shl rh {count}</tt> will perform:<tt>rh = rh << count</tt>_
_ <tt>shr rl {count}</tt> will perform:<tt>rl = rl >> count</tt>_

_ Addresses are integers in the range [0,3]. Example: "<tt>ldh 3</tt>"._

_I heard that if you load 146 in <tt>rl</tt> and 69 in <tt>rh</tt> something special may happen... What are you waiting for? Connect to  port 3786 on 10.13.3.134 and start your limited trial of this amazing  CPU!_

**Solution: "thisisadvancedtechnology**"

* * *
Challenge 19: (50 pts)

_22h 29m 40s -20 50' 20''_

**Solution: "Helix Nebula**"

* * *
Challenge 23: (350 pts)

_ Analyze the following trace and find any revelant information about bank account._

(required file: [network1.pcap](http://www.ictf2011.info/files/network1.pcap))

- extraced zip files from packet capture
- extraced recursive compressed archives until the current file cant be extracted no more
--> file contained a string: xAccount: 499550439979-125084150537

**Solution: 499550439979-125084150537**

* * *
Challenge 24: (50 pts)

_Tweet a picture of your team to @UCSBiCTF using the hashtag #ictf2011_

**Solution: https://twitter.com/#!/pw_sys/status/142648007198904321**

* * *
Challenge 27: (200 pts)

_The OSB company is actually sued for taking part in a weapon traffic across the Atlantic. They want to eliminate the main witness in this case but they still don't know where he is hidden. You have intercepted this audio file from the plaintiff to the judge. Can you find something interesting in there ?_

(required file: [phonecall.wav](http://www.ictf2011.info/files/phonecall.wav))
- file says stereo but audio is only on left track
- audio stops ~1sec before end of track
- right track is text encoded in audio spectrum, not very readable

**Solution: "FONKY STEADY BLD**"

* * *
Challenge 35: (100 pts)

_so many operations_

(required file: [smo](http://www.ictf2011.info/files/smo))

Branfuck code which generates the following Morse Code:

-.-- --- ..-   ..-. --- ..- -. -..   .. - ---...   -- .- -.-- - .... . ..-. .-.. .- --. -... . .-- .. - .... -.-- --- ..-

resolves to: YOUFOUNDIT:MAYTHEFLAGBEWITHYOU
**Solution: "MAYTHEFLAGBEWITHYOU"**

* * *
Challenge 37: (250 pts)

_Money mules keep a low profile. So does this flag. But there is no place to hide._

(required file: [soyouthinkyoucanwebhack2.html](http://www.ictf2011.info/files/soyouthinkyoucanwebhack2.html))

(managed to break the javascript in eclipse)

**Solution: didyouusewepawetagain?**

* * *
Challenge 40: (100 pts)

_On July 4th, 2011 the Twitter account of @foxnewspolitics got  compromised. Well, either that or they made a very macabre joke when  announcing that: "BREAKING NEWS: President @BarackObama assassinated, 2  gunshot wounds have proved too much. It's a sad 4th for #america.  #obamadead RIP". For our security team to close in on the people who  broke into this account, we need to know the exact UNIX timestamp  (seconds precision) of when this Tweet was sent. The file contains data  as we collected it from the Twitter streaming API during that day. Get  back to us with the right answer and we shall provide you with some  CHFs._

(required file: [twitter.gz](http://www.ictf2011.info/files/twitter.gz))

UNIX Timestamp extracted from the .json file.

**Solution: 1309760672**

* * *
Challenge 41: (100)

_Life, The Universe, and Waldo. As everyone knows, a man is defined by the hat he wears. So, where's Waldo's?_

(required file: [FindWaldo.jpg](http://www.ictf2011.info/files/FindWaldo.jpg))

Small manikin inside the large picture. The solution is the coordinates of the white pixel of Waldo's hat.

**Solution: 2155 1912**

* * *
Challenge 43: (75 pts)

(required file: [WhatAmI.code](http://www.ictf2011.info/files/WhatAmI.code))

The link leads to a whitespace program which produces a copy of its own source code.

**Solution: "Quine"**

* * *
Challenge 44: (150 pts)

_ what does it spit out?_

(required file: [wdiso.tar.gz](http://www.ictf2011.info/files/wdiso.tar.gz))

The tar contains 3 files.
0x12 and 0x13 are "encrypted" with rot24.

0x12 is a wired recipe, 0x13 is a poem.
0x90 is a zip bomb.

Use the Chef Compiler to compile the repice into a perl script, which produces the solution.

**Solution: DBLRAINBOW**

* * *
Challenge 46: (125 pts)

_Hey dude, I just found Alexey "Donkey" Dragunov passed out in the server room, stinkin' drunk. Damn him... He probably freaked out for tomorrow, thinking we will never make it. But *we* will. We always do. I still need to "do the deed" with Monaco's tranche. I know the site to use for it is legitimatebiz.ictf2011.info, but I have no freakkin' clue on what to do there. You're good at this stuff. Can you help me? The only thing I found is the sheet of paper attached. It was sticking on the servers, sucked it by the fans. As always, you will *not* fail me._

(required file: [whereismycut.jpg](http://www.ictf2011.info/files/whereismycut.jpg))

The file show a QR code which resolves to:

_otpauth://totp/donkey@ip-172-19-1-77?secret=JUSH3O2LQ3WSJKSC_

otpauth: one time password
ssh donkey@legitimatebiz.ictf2011.info
Verification code: JUSH3O2LQ3WSJKSC
Password: GimmieMyCut
    Welcome to Ubuntu 11.10 (GNU/Linux 3.0.0-12-virtual x86_64)
    
     * Documentation:  https://help.ubuntu.com/
    
    System information as of Fri Dec  2 19:33:14 UTC 2011
    
    System load:  0.0               Processes:           94
    Usage of /:   27.8% of 4.92GB   Users logged in:     0
    Memory usage: 4%                IP address for eth0: 172.19.1.77
    Swap usage:   0%
    
    Graph this data and manage this system at https://landscape.canonical.com/
    Get cloud support with Ubuntu Advantage Cloud Guest
    http://www.ubuntu.com/business/services/cloud
        ___   _        __   __   ____            __
       / _ \ (_)___ _ / /  / /_ / __/____ ___ _ / /___
      / , _// // _ `// _ \/ __/_\ \ / __// _ `// // -_)
     /_/'_'/_/ \_, //_//_/\__//___/ \__/ \_,_//_/ \__/
              /___/
    
    Welcome to a managed virtual machine brought to you by RightScale!
    
    ********************************************************************
    ********************************************************************
    ***       Your instance is now operational.                      ***
    ***       All of the configuration has completed.                ***
    ***       Please check /var/log/messages for details.            ***
    ********************************************************************
    ********************************************************************
    ComeOnTooEasyConnection to legitimatebiz.ictf2011.info closed.

**Solution: "ComeOnTooEasy"**

* * *
Challenge 50: (5 pts)

_1b13dab9c3930f95dc90d9bd4c3a5065_

"Unhash" the MD5 hash.

**Solution: "firstblood"**

* * *
Challenge 51: (25 pts)

_Huaiks are esay, but steoemims tehy don't mkae snese._

"haikus are easy, but sometimes they dont make sense."

**Solution: "Refrigerator"**

* * *
Solutions of challenges of other groups:

[Blogspot](http://antoxar.blogspot.com/2011/12/write-up-mailgw-ictf2011.html) (Main Challenge: Mail Gateway)

[0x1BADFEED](http://blog.lucainvernizzi.net/2011/12/ictf-2011-challanges-writeup.html) (Challenges 14, 46, 13, 9)

[Leet More](http://leetmore.ctf.su/wp/the-significant-game-but-epic-fail-2th-on-ictf-2011/) (Challenges [29](http://leetmore.ctf.su/wp/ictf-2011-challenge-29-800/), [30](http://leetmore.ctf.su/wp/ictf-2011-challenge-30-500/), [31](http://leetmore.ctf.su/wp/ictf-2011-challenge-31/), [32](http://leetmore.ctf.su/wp/ictf-2011-challenge-32/), [33](http://leetmore.ctf.su/wp/ictf2011-challenge-33-100/))

[Délimiteur de données](http://delimitry.blogspot.com/2011/12/ictf-2011-challenge-12-writeup.html) (iCTF 2011 Challenge 12 Writeup)

[Michael @ UCSB SecLab](http://mweissbacher.com/blog/2011/12/20/ictf-2011-challenge-15-writeup) (iCTF 2011 challenge 15 writeup)