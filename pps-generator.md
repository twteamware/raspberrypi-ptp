# GPIO based PPS generator for Raspberrypi3

This document explains how to add the capability of generating a PPS signal on
a GPIO pin of Raspberrypi3. It involves tasks similar to those described in
[README.md](README.md), namely patching configuring and compiling a new kernel
for the Raspberrypi. Hence the thereby described preliminary operations and
default kernel build procedures are assumed.

Again, refer to the official [kernel building raspberrypi.org
page](https://www.raspberrypi.org/documentation/linux/kernel/building.md) for
more details or troubleshooting informations.

### Apply required patches

GPIO PPS generator is implemented by a kernel module. Required patch can be
found in [patches/](patches/).  You can apply them with the `patch` command or
with a git workflow in a separate branch as follow. Run these commands from
within the linux kernel sources directory.

```
git checkout -b pps-generator-patches
git am <path-to-this-repo>/patches/0001-pps-add-gpio-PPS-signal-generator.patch
git am <path-to-this-repo>/patches/0002-add-DT-overlay-for-pps-gen-gpio-generator.patch
```

### Apply required configuration changes

The PPS generator driver must be enabled within the kernel configuration.  Here
`make bcm2709_defconfig` is assumed to be run at least once to bootstrap
default configuration. Run these commands from within the linux kernel sources
directory.

```
echo 'CONFIG_PPS_GENERATOR_GPIO=y' >> .config
```

### Build kernel with PPS generator changes

It is now possible to build the kernel with the changes we made:

```
cd linux
KERNEL=kernel7
make bcm2709_defconfig
make olddefconfig
make -j4 zImage modules dtbs
```

The new kernel image, modules and dtbs must be installed over the current ones.

```
sudo make modules_install
sudo cp arch/arm/boot/dts/*.dtb /boot/
sudo cp arch/arm/boot/dts/overlays/*.dtb* /boot/overlays/
sudo cp arch/arm/boot/dts/overlays/README /boot/overlays/
sudo cp arch/arm/boot/zImage /boot/$KERNEL.img
```

The RaspberryPi is now ready to run with PPS generator enabled on GPIO pin 18.

## Change default GPIO pin for PPS signal

Which pin is used for the PPS signal can be changed in
`arch/arm/boot/dts/overlays/pps-gen-gpio-overlay.dts`. If you need to change
this value, you then need to run `make dtbs` and install the new overlay as
explained above.
