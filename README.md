# onyx-toolchain — the `aarch64-onyx-elf` cross toolchain of Onyx

Prebuilt binaries (Linux x86_64 hosts, WSL included) and the complete corresponding sources of
the toolchain that builds POSIX and C++ programs for [Onyx](https://github.com/stephaneweg/Onyx),
the homemade operating system for the Raspberry Pi 4: GCC 14.2.0 + binutils 2.43 + newlib 4.4,
thread model `posix`, native TLS, newlib's `errno` and `_reent` per thread, libgcc and libstdc++
built against Onyx's `<pthread.h>` (libonyxposix). Onyx's kernel and its existing apps keep Arm's
`aarch64-none-elf`; this one is for programs on libonyxposix and the third-party ports (Onyx's
`docs/03-DEVELOPER-GUIDE.md` §1.1, `docs/POSIX-PLAN.md` "WP-TC resolutions").

## Layout

```
aarch64-onyx-elf-14.2/
    aarch64-onyx-elf-14.2-linux-x86_64.tar.xz.part-00 ...   the tarball, split (split -b 90M -d -a 2)
    SHA256SUMS                                              the whole tarball's sha256, and each part's
    BUILDINFO.txt                                           versions, sources, configure options
sources/aarch64-onyx-elf-14.2/
    binutils-2.43.tar.xz  gcc-14.2.0.tar.xz  newlib-4.4.0.20231231.tar.gz
    gmp-6.3.0.tar.xz  mpfr-4.2.1.tar.xz  mpc-1.3.1.tar.gz   the upstream release tarballs, unmodified
    gcc-14.2.0-libstdcxx-onyx.patch                         the one GCC change (libstdc++'s configure)
    build-onyx-toolchain.sh                                 the script that built the binaries
    overlay/                                                the libonyxposix headers installed into it
    SHA256SUMS
```

The tarball unpacks to `aarch64-onyx-elf-14.2/` (relocatable; Onyx looks in
`/opt/toolchains/aarch64-onyx-elf-14.2`, or wherever its `bin/` is on the `PATH`).

## Install

From an Onyx checkout (Linux, or the WSL shell on Windows):

```sh
sh tools/toolchain/fetch.sh            # shallow sparse clone of this repository, sha256 checked,
                                       # unpacked into /opt/toolchains (PREFIX=<dir> elsewhere)
```

By hand:

```sh
git clone --depth 1 https://github.com/stephaneweg/onyx-toolchain && cd onyx-toolchain/aarch64-onyx-elf-14.2
cat aarch64-onyx-elf-14.2-linux-x86_64.tar.xz.part-* > /tmp/tc.tar.xz
grep ' aarch64-onyx-elf-14.2-linux-x86_64.tar.xz$' SHA256SUMS   # compare with: sha256sum /tmp/tc.tar.xz
sudo mkdir -p /opt/toolchains && sudo tar -C /opt/toolchains -xJf /tmp/tc.tar.xz
/opt/toolchains/aarch64-onyx-elf-14.2/bin/aarch64-onyx-elf-gcc -v      # ... Thread model: posix
```

## Rebuild it from the sources

```sh
sudo apt install build-essential m4 xz-utils curl patch
sh tools/toolchain/build-onyx-toolchain.sh        # in an Onyx checkout (it needs the overlay headers)
```

About 30 minutes on 4 cores (a cross compiler: no bootstrap), ~10 GB of build space in
`~/.cache/onyx-toolchain` (keep it on the Linux side under WSL), 240 MB installed. The script
downloads the same tarballs (pinned URLs and sha256: put this repository's `sources/` files in
`~/.cache/onyx-toolchain/dl/` to build offline), applies the patch, installs the overlay, and
prints the acceptance checks (`Thread model: posix`; `_GLIBCXX_HAS_GTHREADS`,
`_GLIBCXX_USE_CLOCK_MONOTONIC`, `_GLIBCXX_USE_NANOSLEEP`, `_GLIBCXX_USE_SCHED_YIELD`,
`_GLIBCXX_USE_PTHREAD_COND_CLOCKWAIT`, `_GLIBCXX_HAVE_TLS` in `c++config.h`; native TLS).

## Licences

- **GCC, binutils**: GPL-3.0-or-later. **GMP, MPFR, MPC** (linked into `cc1` / `cc1plus`):
  LGPL-3.0-or-later. **newlib**: a collection of BSD-style licences (`COPYING.NEWLIB` in its
  tarball). The complete corresponding source of these binaries is `sources/aarch64-onyx-elf-14.2/`
  in this repository, next to the binaries (GPL-3.0 §6(d)): the upstream tarballs, the patch, the
  build script and the overlay headers.
- **libgcc, libstdc++, libsupc++** (linked into the programs this toolchain compiles):
  GPL-3.0-or-later **with the GCC Runtime Library Exception 3.1** — a program compiled with this
  GCC may be distributed under any licence; using the toolchain imposes nothing on Onyx's programs.
  newlib's licences ask for their notices to accompany binaries that contain it.
- Onyx's own files here (`build-onyx-toolchain.sh`, the text of the patch, the overlay headers
  from libonyxposix, this README): MIT.
