# Solaris patches on FS-UAE 3.1.66

Welcome,

the purpose of these patches is getting FS-UAE 3.1.66 compiled on Solaris based distributions.

The patches and testing has been done on #tribblix an Solaris/OpenSolaris/Illumos based x86 operating sytem.

# Limitations:

Due to not implemented exception handling for Solaris in fs-uae, the jit runtime of fs-uae cannot be used on Solaris based operating systems (please feel free to have a look on src/jit of the fs-uae package and add the missing capabilities :-)) You have to exclude jit in your ./configure parameters

./configure --disable-jit

Dependencies:

fs-uae 3.1.66 requires at least (I assume OpenGL and several other libs as de facto installed but ./configure will tell you what you are missing)

msgfmt sdl2 openal libmpeg2 libmpeg2convert (you can use the libmpeg2 sources within the fs-uae package to build the missing libs)

which should usually be part of either a package or overlay of your Solaris distribution.

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

# Building fs-uae for Solaris 

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




