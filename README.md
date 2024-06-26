```
  ____   ___    __  _      _                  
 |___ \ / _ \  / / | |    (_)                 
   __) | (_) |/ /_ | |     _ _ __  _   ___  __
  |__ < > _ <| '_ \| |    | | '_ \| | | \ \/ /
  ___) | (_) | (_) | |____| | | | | |_| |>  < 
 |____/ \___/ \___/|______|_|_| |_|\__,_/_/\_\
                                              
Linux for i386 machines
```

386linux is a single-user, source-based (but binary build is provided) Linux distribution with tiny but current user space (`busybox` + `musl`).
Main target is a real i386 net-booted machines with at least 8 MB RAM (should work even with 4 MB RAM - and it should work when net-boot is out of a game).

Used Linux kernel is 3.7.10 which should be last kernel directly support i386, however in reality some patching is needed to compile it. So I think that no one ever built/tested this kernel version for/on real i386.

Additional patches & features:

- make kernel compilable - avoid some hard coded `cmpxchg` in kernel source code. 
- [cmpxchg](https://en.wikipedia.org/wiki/Compare-and-swap), xadd, bswap instruction emulator/simulator - currently I'm not able to force gcc to not emit them. Seems like lack of `cmpxchg` instruction is one of reasons why Linux dropped support for real i386.
- Skip `endbr32` instruction for user space - distributions started to use [CET](https://www.intel.com/content/dam/develop/external/us/en/documents/catc17-introduction-intel-cet-844137.pdf) for gcc and even for libraries. `endbr32` is handled as a multinop on i686 generations of CPUs (Pentium II for example) but they are not supported on real i386 (result is CPU exception which is handled by patch - not reproducible in `QEMU`). You can recompile `gcc` without support of `CET` which is one of reasons why [Nix](https://nixos.org/) is involved.
- Fix floppy issues - patches from Linux kernel 5.17 [git](https://lore.kernel.org/lkml/045df549-6805-0a02-a634-81aca7d98db5@linux.com/T/).
- Experimental support for ontrack disk manager partitions - ontrack moves real FAT to a different offset.
- Provide `Nix` based build environment which allows build of Linux kernel with old `gcc` and `perl`.

Main goal of this tiny distribution is to be able to boot via network and be used to:

- Deployment of preconfigured DOS disk images - you can prepare basic setup on virtual machine then move it on real hardware.
- Play around with old hardware.
- As a proof that, with some effort, it works & to learn some old/new stuff :).

Currently, it's not possible to run gray386linux on i486 machines (but it's possible to run gray386linux on Cyrix 486DLC and similar CPUs).
Are you interested in Linux for i486 machines? Take a look at [gray486linux](https://github.com/marmolak/gray486linux). 
i486 machines are still able to run actual Linux kernel (2024).

NOTE FOR Pocket386 USERS!
=========================

386linux runs on M6117 (386SX SOC), which is the heart of Pocket386.
However, you need to add `i8042.dumbkbd=1` to the kernel command-line interface to disable the handling of
keyboard LEDs in order to make the internal keyboard work.


How to build a 386linux
============================

**1.** Decide how to build
---------------------------

If you want just default build, and you have installed `make` and `bash` then
you can just type `make` in `src` directory. You are done and `GOTO 100`.
But if you don't like `make`, you can just type `./graybuild.sh` in `src`
directory.
In both cases, results will be placed in `results` directory.

In case you want to make some changes then follow next steps.


**2.** Get num of cpus
-----------------------

Nix-shell scripts take handle of it.

If you don't want to use `nix-shell`, just type:

`export GR_CPUS=$(nproc --all)`


**3.** Install kernel headers
------------------------------

`make headers_install ARCH=i386 INSTALL_HDR_PATH=../gray386/`


**4.** Leave nix-shell
-----------------------

`exit`


**5.** Build user env with `nix-shell`
---------------------------------------

`cd src/build-env-with-nix/`

`nix-shell --pure`


**6.** Update `.config` in kernel directory
--------------------------------------------

There is mapping hidden in `.config` and you need to change it
to UID of user under you compile a kernel.

`CONFIG_INITRAMFS_ROOT_UID=1001`

`CONFIG_INITRAMFS_ROOT_GID=1001`

Then you can do some other changes with:

`make -j"$GR_CPUS" ARCH=i386 nconfig`


**7.** In Linux kernel directory
---------------------------------

(optional) `make -j"$GR_CPUS" ARCH=i386 nconfig`

`make -j"$GR_CPUS" ARCH=i386 bzImage`

Results are in:

`arch/x86/boot/bzImage`
`usr/initramfs_data.cpio.gz`


**8.** Exit nix-shell
----------------------

`exit`


**9.** (optional) Test build
-----------------------------

You need to have qemu installed.

In kernel directory just run

`./test-build.sh`


**10.** Cleanup build
---------------------

`git clean -f -d -X`

**11.** END
------------
