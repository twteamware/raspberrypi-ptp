# PTP on Raspberrypi3 with Raspbian

This guide explains how to enable support for Precision Time Protocol (PTP) to
a Raspberrypi3 running the Raspbian operating system.

This notes were tested with Raspbian Stretch Lite (2017-11-29). The following
type of tasks are required to have a working PTP client on the Raspberry:

* linux kernel must be patched, configured, built and deployed to the target
* additional packages must be installed and configured
