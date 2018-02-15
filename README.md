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

## Rebuild the kernel to enable PTP support

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

This will build the kernel in its default configuration and will take a *long*
time. If anything fails, up to here please see the
[kernel building raspberrypi.org official page](https://www.raspberrypi.org/documentation/linux/kernel/building.md)
