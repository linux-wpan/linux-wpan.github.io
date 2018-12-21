---
layout: page
title: Roadmap
---

Linux-wpan is a small project mostly developed by two people in their spare
time. Pace is slow and we can not commit to any specific timelines on this
roadmap.

* ~~Wpan-tools 0.9 release with wpan-hwsim~~
* ~~Launch re-designed website~~
* Finish intial ieee802154 and 6lowpan support for kselftest
* Finish kernel documentation rework to new rst style
* New frame parsing style in mac802154 and ieee802154 based on mac80211 frame parsing design. Draft is [mac802154 rx](https://github.com/linux-wpan/linux-wpan-next/blob/wpan_rework_rfc/net/mac802154/rx.c).
* Remove cb context from dev\_hard\_header and introduce generic header generation functions like [header_ops](https://github.com/linux-wpan/linux-wpan-next/blob/wpan_rework_rfc/net/ieee802154/header_ops.c#L80).
* Devicetree extended addr setting, draft is here [ieee802154: add usual way to get extended address via device tree](http://www.spinics.net/lists/linux-wpan/msg01503.html)
* Cleanup/fix 802.15.4 af raw/dgram socket code.
* "node dump" functionality with all neighbour nodes and their last LQI value, addresses, etc. information.
