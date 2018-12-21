---
layout: page
title: Documentation
---

If the hardware and driver are configured correctly your system will come up
with a node type wpan interface, most likely called wpan0.

## Enable 6LoWPAN

Set some valid pan_id, 802.15.4 default is 0xffff which means not assigned:

```
iwpan dev wpan0 set pan_id 0xbeef
```

Currently you need to setup the 6LoWPAN interface manually:

```
ip link add link wpan0 name lowpan0 type lowpan
```

That’s it! Now you have some lowpan0 interface with a 6LoWPAN 1280 MTU which runs on top the wpan interface. As default you have a default link-local address based on the MAC (extended address).

## Setup a 6LoWPAN test network

Let’s assume you want to setup a 6lowpan test network of six nodes. Each of them will be created in their own net namespace to avoid local IPv6 optimizations. Each netns is named according to the wpan interface which available in the net namespace.


```shell
#!/bin/sh

# we need some Private Area Network ID
panid="0xbeef"
# number of nodes
numnodes=6

# include the kernel module for a fake node, tell it to create six
# nodes
modprobe fakelb numlbs=$numnodes

# initialize all the nodes
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
```

Now let us send some data over our network:

```
ip netns exec wpan0 wireshark -kSl -i lowpan0 &
# ping all nodes
ip netns exec wpan0 ping6 ff02::1%lowpan0
```

Now watch wireshark and all the nice ICMP packets there.

## Troubleshooting

If you have issues with a transceiver connected to a SPI bus, double and triple check first if the wiring is correct and the SPI controller is properly configured. Try to decrease SPI clock frequency. Messages like these ones in dmesg output could indicate problems with SPI connection:

at86rf230 spi32765.0: unexcept state change from 0x01 to 0x08. Actual state:
0x01
WARNING: CPU: 0 PID: 61 at drivers/net/ieee802154/at86rf230.c:696 0xc0442644
received tx trac status 4

Also check if the driver does actually generats interrupts on received packets
by checking /proc/interrupts.

Most of the problem reports we are getting turn out to be related to bad cabling
or wrongly setup device tree configurations.

## Sniffing

To sniff traffic first remove all wpan interfaces sitting on top of the wpan phy. You will get a list of all current running phy interface with:

```
iwpan dev
```

then delete all interfaces with:

```
iwpan dev $IFNAME del
```

Finally create a monitor interface:

```
iwpan phy phy0 interface add monitor%d type monitor
```

Then bring this interface up and run wireshark, tcpdump, etc… on it. Running a wireshark as sniffing frontend on the $HOST and connect to a $TARGET like Raspberry Pi which have some ethernet ssh connection, you can try:

```
ssh root@$TARGET 'tshark -i monitor0 -w -' | wireshark -k -i -
```

This will pipe all sniffing frames from $TARGET side to $HOST running wireshark application. Note you need maybe to change the "$TARGET" and "montior0" to your settings.
