# FS-UAE 3.1.66 on Solaris 11.4 and illumos

Patches to build [FS-UAE](https://fs-uae.net/) 3.1.66, the Amiga emulator,
on Oracle Solaris 11.4 (x86) and the illumos distributions (OpenIndiana,
Tribblix, OmniOS).

The same patches are in [oi-userland](https://github.com/OpenIndiana/oi-userland/tree/oi/hipster/components/encumbered/fs-uae)
as the OpenIndiana package `emulators/fs-uae`, and in a solaris-userland
style component for Solaris 11.4 (IPS package `emulators/fs-uae`).

## What is here

| Path | What |
|---|---|
| `patches/` | the patch series, in order, all `-p1` from the top of the fs-uae source tree |
| `dist/solaris/` | the packaging scripts `configure.ac` refers to (copied from `dist/linux`) |

| Patch | Why |
|---|---|
| 0001-configure-ac-solaris | `configure.ac`: a Solaris/SunOS case (`-DSOLARIS`) and `dist/solaris/Makefile` |
| 0002-makefile-am-msgfmt | `Makefile.am`: `msgfmt -o file -v`, the argument order Solaris' own msgfmt accepts |
| 0003-sysdeps | `uae_s8` as `int8_t`: Solaris' `int8_t` is plain `char`, not `signed char`, and the two must be the same type |
| 0004 … 0012 slirp | slirp's headers use an old `_P((...))` prototype macro; Solaris' `<ctype.h>` defines `_P` itself, so the prototypes are written out |
| 0013-bsdsocket-fionread | `FIONREAD` is in `<sys/filio.h>` on Solaris and illumos |
| 0014-endian-solaris | Solaris 11 has no `<endian.h>`: `le16toh()` and friends from `<sys/byteorder.h>`; illumos has `<endian.h>` and is unaffected |

## Building

You need gcc/g++, GNU make, autoconf, automake, libtool, pkg-config,
gettext (msgfmt), glib2, libpng, zlib, freetype, X11/OpenGL, **SDL2** and
**OpenAL**. On OpenIndiana all of them are packages. Oracle Solaris 11.4 has
no SDL2 and no OpenAL: build them first (SDL2 2.32.10 with
`./configure --disable-rpath --enable-static=no`, openal-soft 1.23.1 with
CMake), as oi-userland's `library/SDL2` and `multimedia/openal` do.
libmpeg2 is not packaged on Solaris either; use the copy inside fs-uae
(`--with-libmpeg2=builtin`).

```sh
gtar xf fs-uae-3.1.66.tar.xz
cd fs-uae-3.1.66
for p in ../fs-uae-solaris/patches/*.patch; do gpatch -p1 < "$p"; done
mkdir -p dist/solaris && cp ../fs-uae-solaris/dist/solaris/* dist/solaris/
autoreconf -fi
PATH=/usr/gnu/bin:$PATH \
CC="gcc -m64" CXX="g++ -m64" \
PKG_CONFIG_PATH=/usr/lib/amd64/pkgconfig \
LDFLAGS="-lsocket -lnsl" \
./configure --prefix=/usr --enable-jit --with-libmpeg2=builtin
gmake -j8
gmake install DESTDIR=/tmp/fs-uae-proto
```

`autoreconf -fi` is required: the patches change `configure.ac` and
`Makefile.am`, and `configure`/`Makefile.in` are regenerated from them.
`-lsocket -lnsl` go into `LDFLAGS`: Solaris keeps the socket API in those
libraries.

## The JIT

The earlier version of this README said the JIT could not be used because
exception handling was missing for Solaris.  That is not so for 3.1.66:
FS-UAE's JIT builds and runs on Solaris 11.4 **without any source change**;
configure with `--enable-jit` instead of `--disable-jit`.

- The JIT's SIGSEGV handler (`src/jit/exception_handler.cpp`) uses the
  generic `ucontext` / `uc_mcontext.gregs[REG_RIP]` path, which Solaris
  provides; configure finds it (`HAVE_STRUCT_UCONTEXT_UC_MCONTEXT_GREGS`).
- The translation cache must lie below 2 GB.  Linux uses `MAP_32BIT`; on
  other systems FS-UAE walks mmap address hints, and Solaris honours them:
  the cache lands at 0x42000000.  (Solaris' own `MAP_LOW32` is no help: it
  only promises below 4 GB.)
- `PROT_EXEC` on mmap'ed memory is allowed (Solaris' NXHEAP/NXSTACK
  protect brk memory and the stack, not explicit mappings).
- configure adds `-fno-pie` / `-no-pie` itself when the JIT is on.

Tested on Solaris 11.4 (11.4.90) under Xvfb with FS-UAE's built-in AROS
Kickstart: A1200 with a 68040, 32-bit addressing, 8 MB fast and 64 MB Z3
RAM, `jit_compiler = 1`: the log shows "JIT: Installing segfault handler",
the translation cache at 42000000-42800000, natmem reserved at 0x80000000
("Z3 REAL mapping. JIT direct compatible"), and the emulation runs until a
clean shutdown.  On illumos the same code path applies but has not been
tested yet.

### How much faster

A boot block that runs 100 million iterations of a five-instruction integer
loop (add, eor, rol, subq, bne), timed between two bytes it writes to the
serial port; Kickstart 3.2 (47.115), A1200 with a 68040, 32-bit addressing,
`uae_cpu_speed = max`, warp mode:

| Machine | FS-UAE | Interpreter | JIT | JIT gain |
|---|---|---|---|---|
| Linux laptop, Xeon E-2176M | 3.2.35 (Debian) | 11.1 - 11.5 s | 3.0 - 3.1 s | 3.7x |
| Solaris 11.4 in VirtualBox, Xeon Gold 6132 host | 3.1.66 with these patches | 92.9 - 95.7 s | 22.5 - 23.1 s | 4.1x |

The JIT gives Solaris the same gain as Linux, about four times the
interpreter's speed on integer code.  The absolute difference between the
two rows is mostly the virtual machine, not Solaris or FS-UAE: a plain C
version of the same loop runs 2-3x slower in that VM than on its own host.

The JIT is used only when a configuration asks for it:

```
jit_compiler = 1
cpu = 68040
uae_cpu_24bit_addressing = false
```

The CPU must have 32-bit addressing: with the A1200's stock 68EC020
(24-bit) the log says `Reverting to "indirect" access, because canbang is
zero` and the JIT runs in its slower indirect mode.

## Running

```sh
fs-uae --amiga-model=A1200 --kickstart-file=kick31.rom \
       --floppy-drive-0=disk.adf --chip-memory=2048 --fast-memory=8192
```

or with a configuration file (`fs-uae my-a1200.fs-uae`); see
https://fs-uae.net/docs/options.

## Changes from the earlier version of this repository

The patches used to ship as `fsuae-solaris-patches.tar.gz` with a README of
manual steps.  They had three faults, fixed here:

- `Makefile.am.patch` was reversed (it turned the fixed file back into the
  original).
- `-lsocket -lnsl` and `dist/solaris` were patched into the generated
  `Makefile.in` only, which the documented `aclocal`/`automake` step
  regenerated -- and the patch had a typo (`lnsl`).  The libraries now go
  into `LDFLAGS`, and `Makefile.in` is no longer patched.
- The endian and `FIONREAD` fixes were only described in prose.  The
  described endian fix, `#define le16toh(x) ntohs(x)`, is wrong on x86:
  `ntohs` converts from big-endian, so it byte-swapped little-endian values
  (zip archives, SCP disk images).  `0014-endian-solaris.patch` uses
  `<sys/byteorder.h>`'s `LE_16()`/`LE_32()` instead.
