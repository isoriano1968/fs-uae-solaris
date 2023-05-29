#  FS-UAE 3.1.66 patches required to run on Tribblix/OpenIndiana/Solaris

** UPDATE: 29.05.2023 
**
** Updated to compile on Solaris 11.4.42 CBE (x86), looking for someone to test on SPARC.

Welcome,

the purpose of these patches is getting FS-UAE 3.1.66 compiled on Solaris based distributions.

The patching and testing has been done on #tribblix (http://www.tribblix.org/), release 0m26 an Solaris/OpenSolaris/Illumos based x86 operating sytem.

# Limitations:

Due to not implemented exception handling for OpenSolaris-based systems  in fs-uae, the jit runtime of fs-uae cannot be used on Solaris based operating systems (please feel free to have a look on src/jit of the fs-uae package and add the missing capabilities :-)) You have to exclude jit in your ./configure parameters

./configure --disable-jit

Dependencies:

fs-uae 3.1.66 requires at least (I assume OpenGL and several other libs as de facto installed but ./configure will tell you what you are missing)

msgfmt sdl2 openal libmpeg2 libmpeg2convert (you can use the libmpeg2 sources within the fs-uae package to build the missing libs)

which should usually be part of either a package or overlay of your Solaris distribution (which in the case of native Solaris 11.4.42 CBE is not the case, you have to source the packages from the respective sites and build yourself).

# Patches:

Within the patches folder you will find the list of patches to get fs-uae up and running on Solaris based operating systems.

Apply these patches directly when you have downloaded the fs-uae sources from https://fs-uae.net/download#source (3.1.66 was the current version when I did this) and BEFORE running ./configure --disable-jit and make

# Howto apply patches

Makefile.am.patch.txt

Makefile.in.patch.txt

configure.ac.patch.txt

belong into the root directory of the fs-uae sources

sysdeps.h.patch.txt

belongs into src/include of the fs-uae sources

ip_icmp.h.patch.txt

debug.h.h.patch.txt

mbuf.h.patch.txt

misc.h.patch.txt

sbuf.h.patch.txt

slirp.h.patch.txt

socket.h.patch.txt

tcp_timer.h.patch.txt

udp.h.patch.txt

belong into src/slirp of the fs-uae sources

Copy the solaris folder into the dist folder of the fs-uae sources.

** Solaris 11.4.42 CBE does not use endian.h but instead relies on byteorder.h so you have to update src/scp.cpp and libfsemu/src/data.c with the following updates:
**
**


# Building fs-uae for Solaris 

Before running configure do a "ac-local", "automake" to re-generate the correct "./configure and make" files then

./configure --disable-jit

make

make install

# Running fs-uae once you have build it there are several command line parameters

./fs-uae 

  --floppy-drive-0=xxxxx
  
  --kickstart-file=xxxxx
  
  --amiga-model=(A500/A1200/A4000) select one of them 
  
  --chip-memory=2048
  
  --fast-memory=8192
  
  --floppy-drive-count=4
  
  --hard-drive-0=(insert your hdf file here mine is PRODRIVE80.hdf) 
  
  
Hope it works for you.

Best

Iggi




