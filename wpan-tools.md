---
layout: page
title: Wpan-tools
---

The configuration interface for the ieee802154 subsystem is exposed over
the nl802154 netlink inteface. It allows to configure various phy and MAC
layer parameters and properties. For a covienient usage we offer a set of
small command line utilities that allow these configurations, wpan-tools.

The tools are written in plain C and only minimal dependencies are needed
(the netlink library [libnl](http://www.infradead.org/~tgr/libnl/)).


These tools contains:

**iwpan** main configuration utility (based on the wireless [iw](http://wireless.kernel.org/en/users/Documentation/iw) tool).

**wpan-ping** ping utility on IEEE 802.15.4 layer

**wpan-hwsim** configuration of the hardware simulator driver

## Download

For the last release check out [releases page](https://github.com/linux-wpan/wpan-tools/releases)
