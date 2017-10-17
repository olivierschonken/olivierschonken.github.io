---
layout: post
title:  "64-bit Pi with gallium 3D"
date:   2017-10-11 19:11:22 +0200
categories: first test update
---
# Background
There was a lot of excitement when the Raspberry Pi 3 was released.  An affordable quad-core Cortex-A53, almost unheard of.
Unfortunately, it would take a while to be able to unlock its true potential, as there would be a lot of work to get the Linux kernel
and distributions to make use of that 64-bit goodness.

As a pet project, a friend and I built ourselves retro style arcade machines. Trying out RetroPi, RecalBoxOs etc. It all worked well
enough, but I was interested in throwing off the broadcom binary blob shackles for the video driver, while at the same time migrating to
a complete 64-bit system.

# Where to start?
As I have done a fair amount of work with Buildroot, I chose it as a starting point for the root filesystem. It would include 
all the necessities, and be tweaked here and there to include the latest Rpi Linux Kernel, and the video subsystem would be based on DRM/KMS driver
combined with MESA and the VC4 gallium driver and OpenGLES libraries. No windowing manager, just raw OpenGLES.

To make it even more interesting, instead of using libretro, or advancemame, I would hack and slash the mamedev source and SDL into the cross-compilation
cauldron, and try to bring everything together with no windowing system/compositor to add extra overhead.

Most of the patches to buildroot to fix selection and compilation of the VC4 gallium drivers with libdrm and SDL2 has been committed to the upstream buildroot
repository, thus it should be smooth sailing in future releases.

# Get source and build
Before trying to build the buildroot source, make sure you have all the required packages installed as specified in the [buildroot manual](https://buildroot.org/downloads/manual/manual.html#requirement)

Get Buildroot source with minor SDL, VC4 drm and mesa patches
```
git clone https://github.com/olivierschonken/buildroot.git -b rpi3-drm

cd buildroot

make raspberrypi3_64_drm_defconfig

make all
```

This will produce a sdcard image that can be written directly to sdcard using dd in linux, or win32diskimager(or similar) in Windows.
Example writing with dd -> sudo dd if=output/images/sdcard.img of=/dev/mmcblk0 status=progress

# Changes from standard buildroot 2017.08
* Add SDL2 DRM/KMS driver
* Fix dependency issues for building libdrm and VC4 with aarch64 compiler
* Add support for building and installing kernel overlays for Raspberry Pi
* Turn on audio and add vc4-kms-v3d dtoverlay to config.txt

# Important parts of the configuration

## The toolchain for building for 64-bit Raspberry Pi3.
A custom buildroot toolchain can be used, but I opted for the Linaro toolchain.
```
Toolchain -> Toolchain Type -> External Toolchain
Toolchain -> Toolchain -> Linaro AArch64 2017.02
```
## Latest Kernel source containing the VC4 DRM driver
```
Kernel -> Custom Git repository -> Enter sha of latest commit from 4.13.y branch 
* e.g. [github](https://github.com/raspberrypi/linux/commits/rpi-4.13.y) 52cf298f815cb319c999849aece79fa12a5c1970 as on 8-10-2017
```
## Graphics packages and drivers selection in buildroot
```
Target packages -> Graphic Libraries and applications -> mesa3d
Target packages -> Graphic Libraries and applications -> mesa3d->Gallium vc4 driver
Target packages -> Graphic Libraries and applications -> OSMesa library
Target packages -> Graphic Libraries and applications -> OpenGL EGL
Target packages -> Graphic Libraries and applications -> OpenGL ES
Target packages -> Graphic Libraries and applications -> OpenGL texture float
Target packages -> Graphic Libraries and applications -> sdl2
Target packages -> Graphic Libraries and applications -> sdl2 -> KMS/DRM video driver
Target packages -> Graphic Libraries and applications -> sdl2 -> OpenGL ES
```

## Graphics test utilities
```
Target packages -> Graphic Libraries and applications -> glmark2
Target packages -> Graphic Libraries and applications -> kmscube
```
