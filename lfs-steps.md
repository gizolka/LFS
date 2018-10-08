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

4. Version check 
  * run the version-check.sh
   $ source version-check.sh
   $ source version-check.sh >> version-output.txt
  * Install missing programs

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

7. Creating $LFS/tools and lfs user
   $ mkdir -v $LFS/tools   
   $ ln -sv $lfs/tools / (creates a symlink to host system)
   $ groupadd lfs (creates a new group lfs)
   $ useradd -s /bin/bash -g lfs -m -k /dev/null lfs (creates a new user lfs whose default shell will be bash, -g adds lfs user to the lfs group, -m creates a home directory for lfs, -k /dev/null prevents coping of files from a sceleton directory)
   $ passwd lfs
   Enter new UNIX password: 
   Retype new UNIX password:
   passwd: password updated successfully
   $ chown -v lfs $LFS/tools
   changed ownership of '/mnt/lfs/tools' from root to lfs
   $ chown -v lfs $LFS/sources
   changed ownership of '/mnt/lfs/sources' from root to lfs
   root@debian:~# su - lfs (login as user lfs where - instructs su to start login shell as opposed to a non-login shell)
   lfs@debian:~$

8. Setting up the Environment
  * While logged in as user lfs:
   $ cat > ~/.bash_profile << "EOF" (puts the next lines of text into the file .bash_profile until we type EOF)
   > exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash (command in the .bash_profile file replaces the running shell with a new one with a completely empty environment, except for the HOME, TERM, and PS1 variables. This ensures that no unwanted and potentially hazardous environment variables from the host system leak into the build environment)
   > EOF
   
   $ cat > ~/.bashrc << "EOF"
   > set +h
   > umask 022
   > LFS=/mnt/lfs
   > LC_ALL=POSIX
   > LFS_TGT=$(uname -m)-lfs-linux-gnu
   > PATH=/tools/bin:/bin:/usr/bin
   > export LFS LC_ALL LFS_TGT PATH
   > EOF

[set +h command turns off bash's hash function, setting the user file-creation mask (umask) to 022 ensures that newly created files and directories are only writable by their owner, but are readable and executable by anyone, LC_ALL variable controls the localization of certain programs, making their messages follow the conventions of a specified country. Setting LC_ALL to “POSIX” or “C” (the two are equivalent) ensures that everything will work as expected in the chroot environment, LFS_TGT variable sets a non-default, but compatible machine description for use when building our cross compiler and linker and when cross compiling our temporary toolchain,  putting /tools/bin ahead of the standard PATH, all the programs installed in Chapter 5 are picked up by the shell immediately after their installation]: # comment to the command above

9. Compilation 
  * echo $LFS -> /mnt/lfs
  * Example binutils
   $ cd $LFS/sources
   $ tar -xvf binutils-2.31.1.tar.xz
   $ cd binutils-2.31.1
   $ mkdir -v build -> mkdir: created directory 'build'
   $ cd build/
   $ ../configure --prefix=/tools --with-sysroot=$LFS --with-lib-path=/tools/lib --target=$LFS_TGT --disable-nls --disable-werror 
   configure: creating ./config.status
   config.status: creating Makefile
   $ cd ..
   $ ./config.guess -> x86_64-pc-linux-gnu
   $ cd build/
   $ make
   $ case $(uname -m) in
   > x86_64) mkdir -v /tools/lib && ln -sv lib /tools/lib64 ;;
   > esac
   mkdir: created directory '/tools/lib'
   '/tools/lib64' -> 'lib'
   $ make install
  * GCC-8.2.0:
   sources$ tar, cd new directory 
   sources/gcc-8.2.0$ for file in gcc/config/{linux,i386/linux{,64}}.h
   > do
   >   cp -uv $file{,.orig}
   >   sed -e 's@/lib\(64\)\?\(32\)\?/ld@/tools&@g' -e 's@/usr@/tools@g' $file.orig > $file
   >   echo '
   > #undef STANDARD_STARTFILE_PREFIX_1
   > #undef STANDARD_STARTFILE_PREFIX_2
   > #define STANDARD_STARTFILE_PREFIX_1 "/tools/lib/"
   > #define STANDARD_STARTFILE_PREFIX_2 ""' >> $file
   >   touch $file.orig
   > done
   'gcc/config/linux.h' -> 'gcc/config/linux.h.orig'
   'gcc/config/i386/linux.h' -> 'gcc/config/i386/linux.h.orig'
   'gcc/config/i386/linux64.h' -> 'gcc/config/i386/linux64.h.orig'
   sources/gcc-8.2.0$ case $(uname -m) in 
   >   x86_64)
   >     sed -e '/m64=/s/lib64/lib/' -i.orig gcc/config/i386/t-linux64 
   >   ;;
   > esac
   build$ ../configure --target=$LFS_TGT --prefix=/tools --with-glibc-version=2.11 --with-sysroot=$LFS --with-newlib --without-headers --with-local-prefix=/tools --with-native-system-header-dir=/tools/include --disable-nls --disable-shared --disable-multilib --disable-decimal-float --disable-threads --disable-libatomic --disable-libgomp --disable-libmpx --disable-libquadmath --disable-libssp --disable-libvtv --disable-libstdcxx --enable-language=c,c++

[I removed --disable-libquadmath from configure since during make it showed an error and asked to include libquadmath headers]: 

[Libraries have been installed in:
   /tools/lib/gcc/x86_64-lfs-linux-gnu/8.2.0/plugin

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the `-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the `LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the `LD_RUN_PATH' environment variable
     during linking
   - use the `-Wl,-rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to `/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.]: 

  * Error: /mnt/lfs/tools/bin/../lib/gcc/x86_64-lfs-linux-gnu/8.2.0/../../../../x86_64-lfs-linux-gnu/bin/ld: cannot find crti.o: No such file or directory
collect2: error: ld returned 1 exit status

Added to .bashrc: 
LIBRARY_PATH=/usr/lib/x86_64-linux-gnu:$LIBRARY_PATH 
export LIBRARY_PATH

Error the same.

Solution: sudo apt install gcc-multilib - didn't work out

  * Glibc-2.28:
    ../configure --prefix=/tool --host=$LFS_TGT --build=$(../scripts/config.guess) --enable-kernel=3.2 --with-headers=/tools/include lib_c_forced_unwind=yes libc_cv_c_cleanup=yes

[Encountered many errors, did: # chown -v -R lfs:lfs $LFS/sources and chown -v -R lfs:lfs $LFS/tools
as means to remove the error source]: 

Error: /mnt/lfs/tools/bin/../lib/gcc/x86_64-lfs-linux-gnu/8.2.0/../../../../x86_64-lfs-linux-gnu/bin/ld: cannot find crt1.o: No such file or directory
/mnt/lfs/tools/bin/../lib/gcc/x86_64-lfs-linux-gnu/8.2.0/../../../../x86_64-lfs-linux-gnu/bin/ld: cannot find crti.o: No such file or directory
collect2: error: ld returned 1 exit status


Solution -  added: ln -s /tools/lib/crt*.o /tools/lib/gcc/x86_64-lfs-linux-gnu/8.2.0

  * gcc-8.2.0 after configure and make following error:

[Configuring in ./gcc
configure: loading cache ./config.cache
checking build system type... x86_64-pc-linux-gnu
checking host system type... x86_64-pc-linux-gnu
checking target system type... x86_64-pc-linux-gnu
checking LIBRARY_PATH variable... contains current directory
configure: error: 
*** LIBRARY_PATH shouldn't contain the current directory when
*** building gcc. Please change the environment variable
*** and run configure again.
]:

  * bison3.0.5:
[Error with make check:
make[3]: Entering directory '/mnt/lfs/sources/bison-3.0.5'
  YACC     examples/calc++/calc++-parser.stamp
  CXX      examples/calc++/calc__-calc++-driver.o
  LEX      examples/calc++/calc++-scanner.cc
  CXX      examples/calc++/calc__-calc++-scanner.o
g++: error: ./examples/calc++/calc++-scanner.cc: No such file or directory
g++: fatal error: no input files
compilation terminated.

Solution:
cp Makefile Makefile.bak
   sed -i '/calc++/d' Makefile
   make check
]: 


