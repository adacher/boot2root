#Writeup 1

First step is to get the ip of the vm.

nmap 172.16.129.0-255

Nmap scan report for 172.16.129.3
Host is up (0.00056s latency).
Not shown: 994 filtered ports
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
80/tcp  open  http
143/tcp open  imap
443/tcp open  https
993/tcp open  imaps

A web page is available a this address.

In order to learn more about other paths we use dirb.

cd Applications/dirb222

dirb http://172.16.129.3:80 wordlists/common.txt

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Thu Apr 29 12:51:26 2021
URL_BASE: http://172.16.129.3:80/
WORDLIST_FILES: wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://172.16.129.3:80/ ----
+ http://172.16.129.3:80/cgi-bin/ (CODE:403|SIZE:288)
==> DIRECTORY: http://172.16.129.3:80/fonts/
+ http://172.16.129.3:80/forum (CODE:403|SIZE:285)
+ http://172.16.129.3:80/index.html (CODE:200|SIZE:1025)
+ http://172.16.129.3:80/server-status (CODE:403|SIZE:293)

---- Entering directory: http://172.16.129.3:80/fonts/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.
    (Use mode '-w' if you want to scan it anyway)

-----------------
END_TIME: Thu Apr 29 12:51:32 2021
DOWNLOADED: 4612 - FOUND: 4


After searching through the results, a page in particular caught our attention.
