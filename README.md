# How to compile netcat for ARM devices

This tutorial presents a quick and efficient way to compile netcat for arm architectures.  
Tested on an ARMv7l 32bits device (Freescale quadri-core cpu). 

This repo also contains an already-compiled netcat for arm.

### Setup crosstool-ng

    $ git clone https://github.com/crosstool-ng/crosstool-ng && cd crosstool-ng
    $ ./bootstrap && ./configure --enable-local
    $ make
    $ ./ct-ng menuconfig

Options to check :
  * Target options > Target Achitecture > select arm
  * Operating System > Target OS > either select one of available kernel or specify a custom tarball path
  * If a custom kernel tarball is specified, set 'Custom Linux version' to the corresponding one (e.g. 3.18.12)
  * C-library > C-library > select the libc you want and its version
  * C-compiler > gcc version > select target gcc version

```
$ ./ct-ng build
```

The cross-compiling toolchain is in `~/x-tools/arm-unknown-linux-gnueabi/bin/`


### Create symlinks for ease-of-use

Everything from your toolchain will look like "arm-unknown-linux-gnueabi-*". To avoid any useless and painful configuration, we will create symlink of each tool in a single dir with 'classical' toolchain names (such as gcc, ldd, objdump, etc).

    $ cd ~
    $ mkdir arm-gcc && cd arm-gcc
    $ for f in ~/x-tools/arm-unknown-linux-gnueabi/bin/*; do ln -s $f $(echo $f | cut -d '-' -f9-); done


### Set PATH environment variable

Then, we just add our symlink dir at the beginning of our PATH, so when the system calls 'gcc', it will first grab our custom toolchain.

    $ PATH=/home/$USER/arm-gcc:$PATH


### Download nmap sources (which contains ncat)

    $ cd ~
    $ git clone https://github.com/nmap/nmap && cd nmap


### Compile netcat for arm target

    $ ./configure --host=arm-unknown-linux-gnueabi --with-pcap=null
    $ cd libpcap
    $ make
    $ cd ../ncat/
    $ make
    $ file ncat 
    ncat: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 3.18.12, not stripped


### (Optional) Recompile ncat statically linked (heavier binary but more likely to run on target)

    $ gcc -o ncat -g -O2 -Wall -static -L../libpcap  ncat_main.o ncat_connect.o ncat_core.o ncat_posix.o ncat_listen.o ncat_proxy.o ncat_ssl.o base64.o http.o util.o sys_wrap.o  ../nsock/src/libnsock.a ../nbase/libnbase.a  -lpcap  -ldl
    $ file ncat
    ncat: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), statically linked, for GNU/Linux 3.18.12, not stripped


### (Optional) Strip ncat to reduce binary size

    $ strip ncat
    $ file ncat
    ncat: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), statically linked, for GNU/Linux 3.18.12, stripped


### Useful links

  * Crosstool-ng	https://github.com/crosstool-ng/crosstool-ng
  * Linux Kernel	https://www.kernel.org/pub/linux/kernel/
  * nmap 		https://github.com/nmap/nmap
  * gcc			https://gcc.gnu.org/releases.html

