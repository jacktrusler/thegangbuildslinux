---
marp: true
theme: gaia
---

# Gnu Era Linux
### Let's build Linux From Scratch Over Dinner(s)
A presentation by Jack and Hayden

---

# What is Linux From Scratch Anyway?
Linux from Scratch is a linux distribution that you build from the source tarballs.

The distribution consists of an install manual written by Gerard Beekmans and a set of packages maintained by Bruce Dubbs.

---
# Why are we doing Linux From Scratch?
* Arch Linux added an install script
* Getting to know the fundamental components of Linux and their interoperation
* Learing about the process of bootstrapping an OS and cross compilation
* The Flex :muscle:

---
# What are the requirements to build LFS?
The base Linux From Scratch install has modest requirements 
* A working OS with a compiler available
* A somewhat modern computer with 4 cores, 8GB of RAM
* Spare Disk or Partitions for the new Linux System

A Beyond Linux From Scratch (BLFS) system will require much more to build modern graphical desktop apps.

---
# How long does it take to build LFS?
### The Standard Build Unit (SBU)
* We need a benchmark package we can use to time the sytem
* The total compilation time of the reference package is 1 SBU
* All other compilations can be expressed relative to this SBU

LFS Uses the binutils package as the reference SBU

---
# Enter Build Chungus IV
A formerly state of the art system:
* Devils Canyon i7
* 24GB of Ram
* 2 Solid State Drives
    * Fedora 38 "Host" system drive
    * Gnu Era Linux "Guest" system drive

Everytime we've destroyed the Host system the hostname is incremented.

---

# The Host System
While you could in theory use any compiler to build Linux From Scratch, the packages are heavily dependent on OSS tools like M4, Bison, etc.

#### Fedora, Centos, and RedHat
```shell
sudo dnf groupinstall "Development Tools"
sudo dnf install texinfo
```
#### Other Systems
LFS provides a script [version-check.sh](https://www.linuxfromscratch.org/lfs/view/11.3/chapter02/hostreqs.html)

---

# The $LFS Variable
### The Build Chungus Killer
We use the $LFS Variable to reference the 