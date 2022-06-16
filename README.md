# Solaris patches on FS-UAE 3.1.66

Welcome,

the purpose of these patches is get FS-UAE 3.1.66 compiled on Solaris based distributions.

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

Apply those patches directly when you have downloaded the fs-uae sources from https://fs-uae.net/download#source (3.1.66 was the current version when I did this) and #before running ./configure --disable-jit and #make








