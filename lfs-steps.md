##########################
# Linux From Scratch 8.3 #
##########################

1. Creating a bootable USB-stick with Slackware/Debian etc.
  * Download slackware64-14.2-install.dvd.iso from a mirror e.g. https://linux.rz.rub.de/slackware/slackware-iso/slackware64-14.2-iso/slackware64-14.2-install-dvd.iso
  * Plug an USB-stick into a computer and create the bootable stick:
   $ fdisk -l or lsblk
   $ sudo umount /dev/sda
   $ sudo if=/home/joanna/Downloads/slackware64-14.2-install.dvd.iso of=/dev/sda bs=4M

2. Install the operation system on your target machine.
  * Put the bootable USB-stick into an USB-input slot
  * Switch on the machine
  * Follow the installation instructions

3. Partitioning
  * Checking listed block devices and list partition tables  
   $ lsblk 
   $ fdisk -l
  * Manipulate disk partition table    
   $ cfdisk /dev/sda or $ fdisk /dev/sda
  * Created following partition tables:
   /dev/sda1 128G 83 Linux
   /dev/sda2 8G   82 Linux swap / Solaris
   /dev/sda3 60   83 Linux bootable

4. Setting the $LFS variable
   $ export LFS=/mnt/lfs
   $ echo $LFS
   /mnt/lfs

5. Mounting and further instructions
   $ mkdir -pv $LFS
   $ mount -v -t ext4 /dev/sda3 $LFS
   $ mkdir -v $LFS/sources
   $ chmod -a+wt $LFS/sources 

[change the directory  permissions for all -a, allow to write +w and make it sticky -t or use 1 when using octal numbers in chmod. Sticky means that only the owner and root can delete or rename a directory or files in that directory]: # Side information regarding permissions

6. Created a github account with all useful files, copied from Download LFS page
  * git clone https://github.com/gizolka/LFS
  * root@debian:/git/LFS# wget --input-file=wget-list --continue --directory-prefix=$LFS/sources
  * root@debian:/git/LFS# cp md5sums $LFS/sources
  * root@debian:/git/LFS# pushd $LFS/sources 
   /mnt/lfs/sources /git/LFS
   root@debian:/mnt/lfs/sources# md5sum -c md5sums
   

[pushd command saves the current working directory in memory so it can be returned to at any time, optionally changing to a new directory, popd command returns to the path at the top of the directory stack. This directory stack is accessed by the command dirs ]: # what does puhsd do





  
 

  
 




