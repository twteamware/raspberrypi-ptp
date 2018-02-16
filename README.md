# PTP on Raspberrypi3 with Raspbian

This guide explains how to enable support for Precision Time Protocol (PTP) to
a Raspberrypi3 running the Raspbian operating system.

This notes were tested with Raspbian Stretch Lite (2017-11-29). The following
type of tasks are required to have a working PTP client on the Raspberry:

* linux kernel must be patched, configured, built and deployed to the target
* additional packages must be installed and configured

## Preliminary operations

* Download Raspbian
* flash it on an SD card
* boot the OS

The following commands should be run from a terminal on the RaspberryPi target:
a local or SSH connection for the Raspbian default user pi is assumed from
now on. Note that SSH server is not enabled by default on Raspbian
distribution.

## (optionally) upgrade the distribution

```
sudo apt update
sudo apt upgrade
```

## Install ptp related packages

```
sudo apt update
sudo apt install ethtool linuxptp
```

## Enable software timestamping in ptp4l configuration

Raspberry Pi ethernet phy does not support hardware timestamping: hence
software emulation must be enable in ptp4l configuration. It can be done by
patching ptp4l configuration file as follows:

```
sed -i -e 's/time_stamping.*$/time_stamping\t\tsoftware/' /etc/linuxptp/ptp4l.conf
```

ptp4l will start automatically at boot, but it fails with current kernel which
misses a patch to the ethernet driver and some required configuration non
enabled by Raspbian by default.

## Build a new kernel with enabled PTP support

In order to build the kernel with required patches and configuration options,
kernel sources must be fetched and some required build tools must be installed.

```
git clone --depth=1 https://github.com/raspberrypi/linux
sudo apt install git bc libncurses5-dev
cd linux
KERNEL=kernel7
make bcm2709_defconfig
make -j4 zImage modules dtbs
```

This will build the kernel in its default configuration and will take a
**long** time. If anything fails, up to here please see the
[kernel building raspberrypi.org official
page](https://www.raspberrypi.org/documentation/linux/kernel/building.md).

### Apply required patches

The ethernet driver used by RaspberryPi which comes with current (4.9.y) kernel
sources (smsc95xx) is missing support for software Tx timestamping feature.
This fix is already available in mainline linux sources, but was not merged yet
into the raspberry kernel sources.

Required patch can be found in
patches/0001-smsc95xx-use-generic-ethtool_op_get_ts_info-callback. You can
apply them with the `patch` command or with a git workflow in a separate branch
as follow. Run these commands from within the linux kernel sources directory.

```
git checkout -b ptp-patches
git am <path-to-this-repo>/patches/0001-smsc95xx-use-generic-ethtool_op_get_ts_info-callback
```

### Apply required configuration changes

Some extra configuration options must be applied to the default
`bcm2709_defconfig`. This can be done as follows. Run these commands from
within the linux kernel sources directory.

```
echo 'CONFIG_NET_PTP_CLASSIFY=y' >> .config
echo 'CONFIG_NETWORK_PHY_TIMESTAMPING=y' >> .config
echo 'CONFIG_PTP_1588_CLOCK=y' >> .config
```

The above commands work if you already built the default kernel before applying
the pathes. Otherwise you can enable them manually with any default kernel
configuration target like `make menuconfig`.

### Build kernel with PTP related changes

It is now possible to build the kernel with the changes we made to support PTP:

```
make olddefconfig
make -j4 zImage modules dtbs
```

The new kernel image, modules and dtbs must be installed over the current ones.
Backing up current files is suggested in case something goes wrong and you want
to rollback.

```
sudo make modules_install
sudo cp arch/arm/boot/dts/*.dtb /boot/
sudo cp arch/arm/boot/dts/overlays/*.dtb* /boot/overlays/
sudo cp arch/arm/boot/dts/overlays/README /boot/overlays/
sudo cp arch/arm/boot/zImage /boot/$KERNEL.img
```

The RaspberryPi is now ready to run ptp4l and can be rebooted for changes to
take effect.
