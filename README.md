# PTP on Raspberrypi3 with Raspbian

This guide explains how to enable support for Precision Time Protocol (PTP) to
a Raspberrypi3 running the Raspbian operating system.

This notes were tested with Raspbian Stretch Lite (2018-10-09), running kernel
4.14.71. The following type of tasks are required to have a working PTP client
on the Raspberry:

* linux kernel must be configured, built and deployed to the target
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
software emulation must be enabled in ptp4l configuration. It can be done by
patching ptp4l configuration file as follows:

```
sed -i -e 's/time_stamping.*$/time_stamping\t\tsoftware/' /etc/linuxptp/ptp4l.conf
```

Less recent kernels required patching the ethernet driver to overcome a
limitation with frame timestamping. With those kernels, ptp4l fails to start
while complaining for missing support for timestamping. If you are running
 older kernels, please either consider upgrading or see
[here](kernel-patching.md) for instructions on patching the kernel and fix
the timestamping feature.

## Build a new kernel with PTP support enabled

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

### Build kernel with PTP enabling changes

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

## Troubleshooting

* As mentioned above, you can use `ethtool` to verify that software Tx
timestamping is available on the ethernet interface. The expected output of the
command should be:

```
pi@raspberrypi:~ $ ethtool -T eth0
Time stamping parameters for eth0:
Capabilities:
        software-transmit     (SOF_TIMESTAMPING_TX_SOFTWARE)
        software-receive      (SOF_TIMESTAMPING_RX_SOFTWARE)
        software-system-clock (SOF_TIMESTAMPING_SOFTWARE)
PTP Hardware Clock: none
Hardware Transmit Timestamp Modes: none
Hardware Receive Filter Modes: none
```

If you don't see `SOF_TIMESTAMPING_TX_SOFTWARE` listed, you might be running an
older kernel which requires patching the timestamping support (see
[kernel-patching.md](kernel-patching.md)).

* ptp4l is automatically started as a systemd service at boot in Raspbian. You
can inspect the service status with `systemctl status ptp4l`. If in the log the
message `failed to create a clock` appears, it likely means the required kernel
configuration bits were not enabled.
