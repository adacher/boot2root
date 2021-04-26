This writeup explains how to become root by changing the booting sequence via GRUB.

In our case, the image of boot2root is mounted on a ubuntu VM using VMware fusion v12.


GRUB enables a user to change the way the OS starts by providing a command line interface.

In order to stop someone from using GRUB, it is possible to configure GRUB to work only for specific user accounts by setting user accounts in the GRUB configuration (username and password).

The access to GRUB is not protected so it is possible to boot the OS as we wish.

When starting the VM, simply press the left SHIFT key in order to access GRUB inside the VMware fusion VM's window.

```
boot:
```

Let's start the OS in a new shell :

```
boot: live init=/bin/bash
```

```
root@BornToSecHackMe:/# whoami
root
```

```
root@BornToSecHackMe:/# id
uid=0(root) git=0(root) groups=0(root)
```

Done!






