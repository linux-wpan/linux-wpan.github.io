linux-wpan
==========

Mailinglist: <[linux-wpan@vger.kernel.org](mailto:linux-wpan@vger.kernel.org)\> [Mailinglist-Archive](http://www.spinics.net/lists/linux-wpan)

IRC: #linux-wpan on irc.freenode.net

wpan-tools
----------

To access the nl802154 netlink inteface and also for checking the network connectivity you will need the wpan-tools.

Dependencies:

*   netlink library [libnl](http://www.infradead.org/~tgr/libnl/).


These tools contains:

**iwpan** based on the wireless [iw](http://wireless.kernel.org/en/users/Documentation/iw) tool.

**wpan-ping** Ping utility on IEEE 802.15.4 level.

### Download

For the last release check out [releases](http://wpan.cakelab.org/releases/) page:

[http://wpan.cakelab.org/releases/](http://wpan.cakelab.org/releases/)

Supported Hardware
------------------

Table 1. Supported 802.15.4 Hardware

Transceiver

Supported

Driver

Bus

Where to buy

adf7242

yes

adf7242

SPI or PMOD

[Analog](http://www.analog.com/en/products/rf-microwave/integrated-transceivers-transmitters-receivers/low-power-rf-transceivers/adf7242.html#product-samplebuy)

at86rf212

yes

at86rf230

SPI

[aliexpress](http://de.aliexpress.com/item/Wireless-transceiver-module-zigbee-module-belt-at86rf212-mcu-chip-780m-module/1757611944.html)

at86rf212b

yes

at86rf230

SPI

[digikey](http://www.digikey.com/product-search/en?x=0&y=0&lang=en&site=us&keywords=ATZB-212B-XPRO)

at86rf230

no

at86rf230?

SPI

at86rf231

yes

at86rf230

SPI

[atben](http://downloads.qi-hardware.com/people/werner/wpan/web/) Out of stock. Mailto linux-wpan and Werner to produce maybe new ones. Or contact [qi-hardware](http://en.qi-hardware.com/wiki/Main_Page).

at86rf233

yes

at86rf230

SPI

[openlabs](http://openlabs.co/store/Raspberry-Pi-802.15.4-radio), [digikey](http://www.digikey.com/product-search/en?x=0&y=0&lang=en&site=us&keywords=ATREB233-XPRO)

atusb

yes

atusb

USB

[atusb](http://downloads.qi-hardware.com/people/werner/wpan/web/) Out of stock. Mailto linux-wpan and Werner to produce maybe new ones. Or contact [qi-hardware](http://en.qi-hardware.com/wiki/Main_Page).

cc2420

no

SPI

[aliexpress](http://de.aliexpress.com/store/product/ZIGBEE-networking-technology-Wi-Fi-CC2420-wireless-transceiver-modules-including-antenna/1383438_2036388706.html)

cc2520

yes

cc2520

SPI

[aliexpress](http://de.aliexpress.com/item/CC2520-wireless-module-ZIGBEE-wireless-sensor-networking-module-with-SMA-external-antenna/1921436550.html)

cc2531

no

USB

[aliexpress](http://de.aliexpress.com/item/free-shipping-FOR-ZigBee-CC2531-USB-dongle-protocol-analysis-port-capture-wireless-keyboard-and-mouse/32270975591.html)

mcr20a

yes

mcr20a

SPI

[NXP](https://www.nxp.com/products/wireless-connectivity/thread/2.4-ghz-802.15.4-wireless-transceiver:MCR20A)

mrf24j40

yes

mrf24j40

SPI

[aliexpress](http://de.aliexpress.com/item/MRF24J40MA-I-RM-MRF24J40MA-I-MRF24J40MA-MRF24J40-MICROC-QFN-Import-original/32223258627.html)

RZUSBstick

experimental, [WIP](http://wpan.cakelab.org/firmwares/rzusb.bin)

atusb

USB

[element14](http://www.element14.com/community/docs/DOC-67532/l/avr-rz-usb-stick-module)

Xbee

no

[out of tree](https://github.com/joaopedrotaveira/linux-rpl/blob/master/mainline-3.12.y/0001-Added-XBee-driver-support.patch)

??

[sparkfun](https://www.sparkfun.com/pages/xbee_guide)

Table 2. Boards with Transceivers

Name

Transceiver

Website

Ci40

ca8210

[kickstarter](https://www.kickstarter.com/projects/imgtec/creator-ci40-the-ultimate-iot-in-a-box-dev-kit)

blixten

cc2520

[jopee](https://jopee.wordpress.com/6lowpan-gateway/)

How-To’s
--------

This section describes various How-To’s. Per default you should have already a node type wpan interface.

### 6LoWPAN

Set some valid pan_id, 802.15.4 default is 0xffff which means not assigned:

iwpan dev wpan0 set pan_id 0xbeef

Currently you need to setup the 6LoWPAN interface manually:

ip link add link wpan0 name lowpan0 type lowpan

That’s it! Now you have some lowpan0 interface with a 6LoWPAN 1280 MTU which runs on top the wpan interface. As default you have a default link-local address based on the MAC (extended address).

### Setup a 6LoWPAN test network

Let’s assume you want to setup a 6lowpan test network of six nodes. Each of them will be created in their own net namespace to avoid local IPv6 optimizations. Each netns is named according to the wpan interface which available in the net namespace.

#!/bin/sh

\# we need some Private Area Network ID
panid="0xbeef"
\# number of nodes
numnodes=6

\# include the kernel module for a fake node, tell it to create six
\# nodes
modprobe fakelb numlbs=$numnodes

\# initialize all the nodes
for i in $(seq 0 \`expr $numnodes - 1\`);
do
        ip netns delete wpan${i}
        ip netns add wpan${i}
        PHYNUM=\`iwpan dev | grep -B 1 wpan${i} | sed -ne '1 s/phy#\\(\[0-9\]\\)/\\1/p'\`
        iwpan phy${PHYNUM} set netns name wpan${i}

        ip netns exec wpan${i} iwpan dev wpan${i} set pan_id $panid
        ip netns exec wpan${i} ip link add link wpan${i} name lowpan${i} type lowpan
        ip netns exec wpan${i} ip link set wpan${i} up
        ip netns exec wpan${i} ip link set lowpan${i} up
done

Now let us send some data over our network:

ip netns exec wpan0 wireshark -kSl -i lowpan0 &
\# ping all nodes
ip netns exec wpan0 ping6 ff02::1%lowpan0

Now watch wireshark and all the nice ICMP packets there.

### Troubleshooting

If you have issues with transceiver connected to SPI bus, check first if wiring is correct and SPI controller is properly configured. Try to decrease SPI clock frequency. Messages like these ones in dmesg output could indicate problems with SPI connection:

at86rf230 spi32765.0: unexcept state change from 0x01 to 0x08. Actual state:
0x01
WARNING: CPU: 0 PID: 61 at drivers/net/ieee802154/at86rf230.c:696 0xc0442644
received tx trac status 4

### Sniffing

To sniff first remove all wpan interface which sits on top of the wpan phy. You will get a list of all current running phy interface with:

iwpan dev

then delete all interfaces with:

iwpan dev $IFNAME del

Finally create a monitor interface:

iwpan phy phy0 interface add monitor%d type monitor

Then bring this interface up and run wireshark, tcpdump, etc… on it. Running a wireshark as sniffing frontend on the $HOST and connect to a $TARGET like Raspberry Pi which have some ethernet ssh connection, you can try:

ssh root@$TARGET 'tshark -i monitor0 -w -' | wireshark -k -i -

This will pipe all sniffing frames from $TARGET side to $HOST running wireshark application. Note you need maybe to change the "$TARGET" and "montior0" to your settings.

Developing
----------

Current developing repository is [bluetooth-next](http://git.kernel.org/cgit/linux/kernel/git/bluetooth/bluetooth-next.git). All patches should be send to <[linux-wpan@vger.kernel.org](mailto:linux-wpan@vger.kernel.org)\> and based on bluetooth-next.

For wpan-tools checkout the [wpan-tools](https://github.com/linux-wpan/wpan-tools) repository. Also send patches to <[linux-wpan@vger.kernel.org](mailto:linux-wpan@vger.kernel.org)\> for it with a "wpan-tools" tag. The same for [wpan-misc](https://github.com/linux-wpan/wpan-misc).

### Open Tasks

*   There is a lot of missing features for enum definition to some string definition in iwpan which can be lookup in 802.15.4 standard. Words say more than numbers…

    *   channel/page to frequency

    *   cca modes/opts

    *   etc


*   Missing features which wireless has and wpan not. Since we based our implementation on wireless we should sync "good patches" from wireless branch.

    *   Whatever you want and find


*   rework

    *   missing features in nl802154, crypto etc.

    *   new frame parsing style in mac802154 and ieee802154 based on mac80211 frame parsing design. Draft is [mac802154 rx](https://github.com/linux-wpan/linux-wpan-next/blob/wpan_rework_rfc/net/mac802154/rx.c). Crypto need to be done at first, otherwise I can’t test it.

    *   remove cb context from dev\_hard\_header and introduce generic header generation functions like [header_ops](https://github.com/linux-wpan/linux-wpan-next/blob/wpan_rework_rfc/net/ieee802154/header_ops.c#L80). Here too, crypto need to be done at first.


*   systemd

    *   add basic functionality for nl802154 and 6lowpan setup in systemd-networkd


*   network-manager

*   devicetree extended addr setting, draft is here [ieee802154: add usual way to get extended address via device tree](http://www.spinics.net/lists/linux-wpan/msg01503.html)

*   RPL? - not our job, need to go through ipv6 netdev community, but we should do something to have a "started" mainline solution.

*   cleanup/fix 802.15.4 af raw/dgram socket code. We should use bluetooth socket code as example.

*   In wireless exists a "station dump", we need something similar "node dump" with all neighbour nodes and their last LQI value, addresses, etc. information.


### Rework

Currently a rework of 802.15.4 subsystem is in progress.

The rework will contains a new netlink API and a general handling about frame creation and parsing.

Table 3. Work status

Milestone

Description

state

1\. nl802154

new netlink framework

mostly done

2\. crypto-layer

accessable over nl802154 and tools

TODO

3\. frame parsing

remove bugs, more generical

TODO

**NOTE** we need to finish rework state 2. before we can start 3 or anything else which touch the frame parsing/generation.

For the nl802154 framework you will need the wpan-tools, older kernels need lowpan-tools but this is not recommended.

### Specifications

*   [IEEE](http://www.ieee802.org/15/pub/TG4.html):

    *   [http://standards.ieee.org/about/get/802/802.15.html](http://standards.ieee.org/about/get/802/802.15.html)
    *   802.15.4-2003
    *   802.15.4-2006
    *   802.15.4a-2007
    *   802.15.4c-2009
    *   802.15.4d-2009
    *   802.15.4-2011
    *   802.15.4e-2012
    *   802.15.4f-2012
    *   802.15.4g-2012
    *   802.15.4j-2013
    *   802.15.4k-2013
    *   802.15.4m-2014
    *   802.15.4p-2014

*   [IETF](https://www.ietf.org/):

    *   [IPv6 over Low-Power Wireless Personal Area Networks (6LoWPANs): Overview, Assumptions, Problem Statement, and Goals](http://tools.ietf.org/html/rfc4919)
    *   [Transmission of IPv6 Packets over IEEE 802.15.4 Networks](http://tools.ietf.org/html/rfc4944)
    *   [Compression Format for IPv6 Datagrams over IEEE 802.15.4-Based Networks](http://tools.ietf.org/html/rfc6282)
    *   [RPL: IPv6 Routing Protocol for Low-Power and Lossy Networks](http://tools.ietf.org/html/rfc6550)
    *   [Neighbor Discovery Optimization for IPv6 over Low-Power Wireless Personal Area Networks (6LoWPANs)](http://tools.ietf.org/html/rfc6775)
    *   [6LoWPAN-GHC: Generic Header Compression for IPv6 over Low-Power Wireless Personal Area Networks (6LoWPANs)](http://tools.ietf.org/html/rfc7400)

*   IETF working groups:

    *   [IPv6 over Low power WPAN (6lowpan)](http://datatracker.ietf.org/wg/6lowpan/documents/)
    *   [Routing Over Low power and Lossy networks (roll)](http://datatracker.ietf.org/wg/roll/documents/)
    *   [IPv6 over the TSCH mode of IEEE 802.15.4e (6tisch)](http://datatracker.ietf.org/wg/6tisch/documents/)
    *   [IPv6 over Networks of Resource-constrained Nodes (6lo)](http://datatracker.ietf.org/wg/6lo/documents/)
    *   [Constrained RESTful Environments (core)](http://datatracker.ietf.org/wg/core/documents/)

*   [6LoWPAN IANA assignments](https://www.iana.org/assignments/_6lowpan-parameters/_6lowpan-parameters.xhtml)


### Changing the Website

If you want to take part in developing this website, do the following:

git clone https://github.com/linux-wpan/wpan-misc.git
cd wpan-misc/website
git checkout -b myproposal
\# Change this page to whatever you like.
$EDITOR index.txt
git commit -s -a
\# be sure to be registered to linux-wpan@vger.kernel.org before sending the
\# patch
git send-email --to linux-wpan@vger.kernel.org -1

Next wait for the reactions on the mailinglist and if your change is integrated.

* * *
