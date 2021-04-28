In this writeup, we use the mount and unsquashfs commands inside a Docker container to access data contained in the iso file of the boot2root vm.

This solution is an alternative starting route than the beginning of writeup1.md and is easier as well.

docker build . -t ubuntu

docker run -ti --privileged ubuntu

root@78efb5af0d86:/home/tmp# sudo mount BornToSecHackMe-v1.1.iso boot2root/ -o loop

root@78efb5af0d86:/home/tmp# sudo unsquashfs -f -d unsquashedfs_dir/ boot2root/casper/filesystem.squashfs

root@78efb5af0d86:/home/tmp/unsquashedfs_dir# cat home/LOOKATME/password
lmezard:G!@M6f4Eatau{sF"

