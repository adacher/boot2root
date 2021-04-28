In this writeup, we use the mount and unsquashfs commands inside a Docker container to access data contained in the iso file of the boot2root vm.

This solution is an alternative starting route than the beginning of writeup1.md and is easier as well.

The first step is to build our Docker image within within which we specify which dependencies to install and we also download the .iso file of the VM.

```
docker build . -t ubuntu
```

Second step is to run the container with --privileged, otherwise, the mount command would not work. It is also not possible to give this instruction directly as a RUN command in the Dockerfile due to permission restrictions and how Docker works.

```
docker run -ti --privileged ubuntu
```

We mount the .iso file inside the boot2root folder, '-o loop' allows us to access the result as regular files and create a "fake" device, which is simply a file.

```
root@78efb5af0d86:/home/tmp# sudo mount BornToSecHackMe-v1.1.iso boot2root/ -o loop
```

Lastly, we get the content of the filesystem.squashfs file inside the unsquashedfs_dir/ directory.

```
root@78efb5af0d86:/home/tmp# sudo unsquashfs -f -d unsquashedfs_dir/ boot2root/casper/filesystem.squashfs
```

The entigrity of the system was rebuilt as read-data only data and we can access it.

```
root@78efb5af0d86:/home/tmp/unsquashedfs_dir# cat home/LOOKATME/password
lmezard:G!@M6f4Eatau{sF"
```

Proceed to writeup1.md
