### Apply required patches

The ethernet driver for RaspberryPi (smsc95xx) which comes with older (i.e.
4.9.y) kernel sources was missing support for software Tx timestamping feature.
This fix has already been available in mainline linux sources for a while.  If
you use a more recent kernel and need to check if this patch is required or
not, you can use the `ethtool` command to verify whether
`SOF_TIMESTAMPING_TX_SOFTWARE` is listed or not. If it's in the list, you can
skip to the next section, otherwise patching is required.

Required patch can be found in
[patches/0001-smsc95xx-use-generic-ethtool_op_get_ts_info-callback.patch](patches/0001-smsc95xx-use-generic-ethtool_op_get_ts_info-callback.patch). You can
apply them with the `patch` command or with a git workflow in a separate branch
as follow. Run these commands from within the linux kernel sources directory.

```
git checkout -b ptp-patches
git am <path-to-this-repo>/patches/0001-smsc95xx-use-generic-ethtool_op_get_ts_info-callback
```

Then proceed with the building instruction at [README.md](README.md)
