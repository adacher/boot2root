FROM ubuntu

RUN apt update && apt upgrade -y
RUN apt install squashfs-tools -y
RUN apt install wget -y
RUN apt install sudo -y
RUN sudo mkdir /home/tmp
WORKDIR /home/tmp
RUN wget https://projects.intra.42.fr/uploads/document/document/2832/BornToSecHackMe-v1.1.iso
RUN mkdir boot2root
RUN mkdir unsquashedfs_dir