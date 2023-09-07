## [Linux From Scratch](https://www.linuxfromscratch.org/lfs/view/stable-systemd/)

## How did we start our journey

### Computer:

2SSDs - 250gb 
24gb ram
i7-4790k

# We will call him: BuildChungus

### Operating system: 

Centos 9 stream

### First steps: 
- Install Centos9 
- Install developer tools to get needed packages
- username: KenBitson 

Some packages not provided, check using bash script in LFS chapter 1
`sudo dnf whatprovides makeinfo` 
`sudo dnf install texinfo`

LFS provides a script [version-check.sh](https://www.linuxfromscratch.org/lfs/view/11.3/chapter02/hostreqs.html)
to check if everything is downloaded right.
### Next: Disk Partition

`lsblk`

Note: Look up efi
Note: Look up partition table GPT  

partion with fdisk

1st partition 128Mb for efi
2nd partition 256Mb for boot
3rd partition remaining for /root/

Create a FAT formatted efi partition
`sudo mkfs.vfat /dev/sdb1 -n EFISYS` 

Create a ext4 formatted main partition and ext4 boot
`sudo mkfs.ext4 /dev/sdb1 -n EFISYS` 

Default is 4k blocks

Making a efi partition is a beyond linux from scratch thing.

make a LFS environment variable `export LFS=/mnt/lfs`
make root and efisys dir and mount them to new partitions

    sudo mkdir              /mnt/lfs
    sudo mkdir              /mnt/lfs/boot
    sudo mkdir              /mnt/lfs/boot/efi
    mount -L GnuEraRoot     /mnt/lfs
    mount -L GnuEraBoot     /mnt/lfs/boot
    mount -L EFISYS         /mnt/lfs/boot/efi

Go into `/etc/fstab`
fstab is what the system uses when it boots to figure out what it should mount.

    LABEL=GnuEraRoot    /mnt/lfs/           ext4 defaults 0 0
    LABEL=GnuEraBoot    /mnt/lfs/boot       ext4 defaults 0 0
    LABEL=EFISYS        /mnt/lfs/boot/efi   ext4 defaults 0 0

mtab to see if drives are successfully mounted

make a sources directory and make it sticky (only owner can delete file within sticky directory)

    mkdir /mnt/lfs/sources/    
    chmod a+wt /mnt/lfs/sources/

## Download all linux packages needed for systemd based LFS install

wget list of systemd packages and put them into the new sources dir

    wget --input-file=wget-list-systemd --continue --directory-prefix=$LFS/sources

speed stuff up with parallels and tmux, to do so we need the Extra Packages for Enterprise (EPEL)

    sudo dnf config-manager --set-enabled crb
    sudo dnf install epel-release epel-next-release  

Make sure everything is installed by rerunning command with no-clobber

    wget -nc --input-file=wget-list-systemd --continue --directory-prefix=$LFS/sources

We had a package endpoint 404, zlib. So we went to https://zlib.net/fossils and downloaded the old 
zlib package zlib-1.2.13, that LFS asked for.

LFS gives you a list of checksums to ensure all packages are installed. Our downloaded Zlib package didn't 
match the checksum, but we downloaded the version asked for so we pressed on.

## Create a limited directory layout on LFS Filesystem

    mkdir -pv $LFS/{etc,var} $LFS/usr/{bin,lib,sbin}

    for i in bin lib sbin; do
      ln -sv usr/$i $LFS/$i
    done

    case $(uname -m) in
      x86_64) mkdir -pv $LFS/lib64 ;;
    esac

We make a sym link from `$LFS/bin $LFS/lib $LFS/sbin` to `usr/bin usr/lib usr/sbin` apparently this is 
standard practice for linux systems. usr/bin is for the majority of tools you use on your system.

## Make a LFS user

make lfs group `groupadd lfs`
note: Check LFS manual for rest of commands and why they used them

## Quick Sidebar

We bricked the system by accident (>^.^)>
Maybe because we uh deleted a bin folder we shouldn't have <(^.^<)

# BuildChungusII 

Fedora desktop this time, and this time... it's for real

Additional software added 

    C Developer tools
    Development Tools

username_redux: kenbitson
pc_name_redux: buildchungusII

there's a script provided in LFS to give you a version check for all of the utils you download on 
your system, with `C Developer tools` and `Development tools` packages we're only missing `makeinfo` which 
is provided by `textinfo` package. 

    sudo dnf whatprovides makeinfo

- dnf install textinfo
- check everything is installed [version-check.sh](https://www.linuxfromscratch.org/lfs/view/11.3/chapter02/hostreqs.html)
- run script from LFS
- make partition

        This time we made a 1Mb BIOS boot partition 
        A 256Mb boot partition
        Rest of the drive is reserved for the filesystem `/`

        all in `ext4`

- Put startup directories in `fstab`
- Reboot
- Check they are mounted with mtab 
- Make a sources directory, make it sticky
- wget all the needed packages, download them fast with parallels
- wget again, pass -nc (no-clobber) flag to make sure everything is correct and doesn't override
- zlib wasn't downloaded from mirror
- downloaded zlib ourselves
- create limited directory layout 
- make symlinks, be careful, easy to frick the whole filesystem up here :>
- mkdir tools and lib64
- make a user
- make bash profile epic
- su lfs

## Starting to cook with binutils and GCC (chapter 5)

- make a build directory in binutils package after unzip binutils package with `tar xf`
- One Standard Build Unit SBU (all of binutils) is 693MB and took this sexy i7 only 2min 25s.
- GCC takes 3 SBU, took us 7min 13s
- GCC limits.h file is incomplete, run command that completes it
- Start installing linux headers `make mrproper`
- Now install GlibC for the C libary
- Make sure the install target matches what LFS wants and DO NOT compile as root, we used LFS user
- check the linker and toolchain are working as expected with a sanity check
- finalize the installation of the limits header file
- Now that we have glibc we can go back to GCC and compile the c++ libaries needed for the rest of the install.

## Cross compiling temporary tools (chapter 6)

- Using machine A we are building temporary tools for machine B which will eventually be used in 
their full context on machine C. Note, all of this is being done on the same hardware, which makes 
this a bit confusing, but the three machines are the 3 separate built filesystems. 
- We made an alias for the repetitive command `make DESTDIR=$LFS install` Make Install Linux From Scratch (milfs)
- We build the library one tool at a time :> using make -j 10 we run parallels to multithread the 
process, speeding it up significantly. Number of cores/threads + 2 seems to give the best performance
- Recompiling the GCC with new flags that include C and C++ takes longer than the original compilation
12m 43s

## Entering Chroot and Building temp Tools

Now the system is isolated from the host environment except for the host kernal needed to run the 
commands on our new linux filesystem.

We need to make the root user have total ownership of the filesystem because the lfs user will not be 
present when we make a new filesystem. Also we want root to own everything anyway. 

## Uh-OH we may have accidentally learned some things

Here is the command that killed BuildChungusII `chown -R root:root $LFS/{usr,lib,var,etc,bin,sbin,tools}`

So uh, LFS variable wasn't set on root user when we ran the command to change ownership. So unfortunately 
we have killed BuildChungusII. RIP.

# BuildChungusIII

Say hello to our newest computer, we lost some real ones, but we press on. 

Go back into `/etc/fstab` and relabel.

    LABEL=GnuEraRoot    /mnt/lfs/           ext4 defaults 0 0
    LABEL=GnuEraBoot    /mnt/lfs/boot       ext4 defaults 0 0
    LABEL=EFISYS        /mnt/lfs/boot/efi   ext4 defaults 0 0

### 7.2
Make sure LFS environment variable is set for sudo and current user
`export LFS=/mnt/lfs/`

or use `-E` to pass users environment variables into sudo before running the command

`sudo -E chown -R root:root $LFS/{usr,lib,var,etc,bin,sbin,tools}`

### 7.3 - 7.4
We have to bind host data systems to be able to use our fully built kernal on the host machine, 
since it is not yet build for GnuEraLinux

    mkdir -pv $LFS/{dev,proc,sys,run}

    mount -v --bind /dev $LFS/dev

    mount -v --bind /dev/pts $LFS/dev/pts
    mount -vt proc proc $LFS/proc
    mount -vt sysfs sysfs $LFS/sys
    mount -vt tmpfs tmpfs $LFS/run

Now we can Chroot into LFS

### 7.5 - 7.6
- Make a bunch of directories
- Make users
- Make Groups
- Make a test user, will be deleted at end of section
- Start new shell to get rid of `I have no name!`
- Make logs files to see info on who logged in and when
- Build temporary tools in order to build other tools that aren't temporary on the system
    - Gettext
    - Bison
    - Perl 
    - Python
    - TexInfo 
    - Util-Linux
- Clean up extra files to save ~1Gb of space
