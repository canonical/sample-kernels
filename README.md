# Snappy Ubuntu Core Kernel Porting Guide

## Introduction

When dealing with 3rd-parties, or simply when trying to port Snappy Ubuntu Core
to a new piece of hardware, the questions that we most frequently face are:

* **Q**: What are the kernel config options that Snappy Ubuntu Core requires to run?
* **Q**: What are the options that the kernel I use on my hardware must support to properly run Snappy Ubuntu Core?
* **Q**: Is feature X mandatory to run Snappy Ubuntu Core?

The following is meant to answer these questions and serve as a guide to
facilitate third parties on non-Ubuntu based kernels and most likely stable
kernel versions (eg. 3.10, 3.14) to enable the minimum kernel requirements in
order to get an introductory experience with Snappy Ubuntu Core. This may also
highlight areas where it would be expected that features/functionality would be
absent/broken due to the kernels lacking the support.

The list of features that Snappy Ubuntu Core requires to work is the sum of all
the features required by the building blocks and software that Snappy Ubuntu
Core builds upon.  With this in mind, the Snappy Ubuntu Core delta configuration
was split in kconfig fragments (one per area / software) that people can apply
on top of the base defconfig of their hardware.


**DISCLAIMER**:  It should also be noted that these branches are NOT officially maintained.
There is no routine security maintenance nor bug fixing done for these.  For
official support and maintenance, please contact <???@???>.

## Kernel Config

*Goal*: Provide a series of config changes that developers can apply with
scripts/kconfig/merge_config.sh on top of their defconfig file. 

The instructions below assume you are using an Ubuntu 16.04 x86_64 workstation,
have a recent version of snapcraft installed (>= 2.8.4), and have the tools
required to build a kernel installed (eg. Ubuntu requires *build-essential*,
*toolchain*, *apt-get build-dep linux-image-\`uname -r\`*, etc.).

###### Overview of the kconfig delta:

An example of the kconfig delta is available [here](https://github.com/leannogasawara/sample-kernels/tree/stable-3.14.y/kernel/configs/snappy) and is composed of:

 1. Generic.config - contains the features that we enforce in the Ubuntu config, and
in general all the configs that ‘make sense’ to enable 
 2. Security.config - security options that we want to turn on - AA, SECCOMP, STACKPROTECTOR, etcetc
 3. Systemd.config - features required by systemd, see also [README -
REQUIREMENTS section](https://github.com/systemd/systemd/blob/master/README)
 4. Snappy.config - features that are required by ubuntu-core go here
 5. Containers.config - features required by lxc/docker, see also [check-config.sh](https://github.com/docker/docker/blob/master/contrib/check-config.sh)

Reference branches:
 * Linux LTS
	* [stable-3.10.y](https://github.com/leannogasawara/sample-kernels/tree/stable-3.10.y)
	* [stable-3.14.y](https://github.com/leannogasawara/sample-kernels/tree/stable-3.14.y)
	* [stable-3.18.y](https://github.com/leannogasawara/sample-kernels/tree/stable-3.18.y)
	* [stable-4.4.y](https://github.com/leannogasawara/sample-kernels/tree/stable-4.4.y)
 * Android LTS
	* [android-3.10.y](https://github.com/leannogasawara/sample-kernels/tree/android-3.10.y)
	* [android-3.14.y](https://github.com/leannogasawara/sample-kernels/tree/android-3.14.y)

All of these branches went under these modifications:

The directory ‘kernel/configs/snappy’ containing the necessary kconfig delta
(generic.config, security.config, snappy.config, systemd.config,
containers.config) was added to the root of every branch
The necessary kbuild bits to make the kernel build using the kconfig fragments
in kernel/config/snappy were cherry-picked from upstream, and
a snapcraft.yaml file was added to the root of every branch, ready to snap the
kernel using snapcraft.

Every one of these branches was tested with the aforementioned snapcraft.yaml,
and it produced a working x86_64 kernel capable of booting a recent ubuntu-core
amd64 image.

People enabling new hardware can base their derivative work out of these
branches following these steps:

Clone the ‘ubuntu-core’ official git tree:
```
git clone https://github.com/leannogasawara/sample-kernels.git
```
Pick the kernel version that suit your needs:
```
cd sample-kernels && git checkout stable-3.14.y
```
If the target hardware requires any custom patch, apply it now on top of this
tree (in case of a big BSP stack, it makes more sense to rebase the BSP on top
of this branch).
If the target hardware requires any custom kernel configuration, either create a
new kconfig fragment in ‘kernel/configs/snappy’ and adjust the ‘kdefconfig’
section in snapcraft.yaml  or apply the missing CONFIG directly in the
‘kconfigs’ section in snapcraft.yaml
If the target hardware uses a different base defconfig then ‘x86_64_defconfig’,
change the first entry in the ‘kdefconfig’ section in snapcraft.yaml
appropriately (e.g. multi_v7_defconfig):
```
kdefconfig: [multi_v7_defconfig, …]
```
Modify the name of the kernel snap in snapcraft.yaml to reflect the target
hardware and build the kernel snap:
```
snapcraft -d
```
Or in case of cross-compilation (e.g. armhf):
```
snapcraft -d --target-arch=armhf
```
