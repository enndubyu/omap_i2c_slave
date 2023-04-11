I<sup>2</sup>C Slave/Multimaster Driver for Linux on TI OMAP/Sitara
----------------

This patch adds multi-master and slave support to Linux's [i2c-omap](https://github.com/torvalds/linux/blob/master/drivers/i2c/busses/i2c-omap.c)
bus driver (for Texas Instruments' Sitara and OMAP SoCs), enabling full I<sup>2</sup>C
support for the BeagleBone Black and many other SBCs/SoMs supported by this driver.

### Contents

<!-- TOC -->

- [Requirements:](#requirements)
- [Building/Installing](#buildinginstalling)
    - [Applying Patch Manually](#applying-patch-manually)
    - [Yocto](#yocto)
- [Usage](#usage)
    - [Slave drivers in mainline kernel](#slave-drivers-in-mainline-kernel)

<!-- TOC -->

<a id="requirements"></a>
## Requirements:

- Kernel Version 4.13+
- KConfig
  + CONFIG_I2C_SLAVE
  + CONFIG_ARCH_OMAP2PLUS [^1]

<a id="buildinginstalling"></a>
## Building/Installing

<a id="applying-patch-manually"></a>
### Applying Patch Manually

First, obtain a copy of the kernel sources from Linus' github or your BSP:
- [https://github.com/torvalds/linux](https://github.com/torvalds/linux)
- [TI Processor SDK](https://software-dl.ti.com/processor-sdk-linux/esd/AM335X/08_02_00_24/exports/docs/linux/Foundational_Components_Kernel_Users_Guide.html#getting-the-kernel-source-code)
- [Yocto devtool](https://docs.yoctoproject.org/2.5/kernel-dev/kernel-dev.html#using-devtool-to-patch-the-kernel)
- [bb-kernel scripts](https://github.com/RobertCNelson/bb-kernel/tree/am33x-rt-v6.1)

Now you can simply clone this repo, apply the patch file from the root of the
kernel source tree, add `I2C_SLAVE` to the `.config`, (re)build, and deploy the
kernel as normal.

```sh
git clone git@github.com:enndubyu/omap_i2c_slave.git
cd linux

# Apply patch
git apply ../omap_i2c_slave/0001-i2c-omap-add-slave-support.patch

# Select I2C_SLAVE=y and any desired backend drivers (e.g. I2C_SLAVE_EEPROM=m)
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig

make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- all
```

<a id="yocto"></a>
### Yocto

You can extend the [kernel recipe](https://git.yoctoproject.org/meta-ti/tree/meta-ti-bsp/recipes-kernel/linux/linux-ti-staging_5.10.bb)
with a `.bbappend` that applies this patch and adds the required options to your
`.config`. See the [yocto manual](https://docs.yoctoproject.org/2.5/kernel-dev/kernel-dev.html#creating-and-preparing-a-layer)
for details.

<a id="usage"></a>
## Usage

Acting as a slave on an I<sup>2</sup>C bus requires slave support in the bus
driver and a backend driver providing the actual functionality. This patch adds
the former. For the backend driver, the kernel already includes a couple of
existing drivers that might serve your needs, but most applications will require
a custom driver. See the [kernel documentation](https://docs.kernel.org/i2c/slave-interface.html)
and the examples listed below for details on how to use the slave interface in
a backend driver.

<a id="slave-drivers-in-mainline-kernel"></a>
### Slave drivers in mainline kernel

- [i2c-slave-eeprom.c](https://github.com/torvalds/linux/blob/master/drivers/i2c/i2c-slave-eeprom.c)
- [ipmb_dev_int.c](https://github.com/torvalds/linux/blob/master/drivers/char/ipmi/ipmb_dev_int.c)

<!-- Footnotes -->

[^1]: Rev. 1 IP (used in first-generation devices such as OMAP1510 and OMAP310) is not supported by this patch.
