# Basic Network Triage in Gottwald

## About the network cards.

Knowledge gained from looking at the lights on the cards is probability rather than facts. 

- If no cord is plugged in, nothing will happen. If the computer has a network connection, it is on wireless.
- If a cord is plugged in and no lights are on, then the card either has no power to it, or it has been disabled in the computer's configuration. 
- If one light is on, it usually means there is an electrical connection to something on the other end of the wire. It doesn't mean the computer that has the network cord is powered on or usable. It is probably powered on, but we have PoE (power-over-ethernet), and the lights could be on because the card is receiving power over the network. Why would this be the case? It's so you can identify ethernet cables that are supplying power as well as data. Most routers and switches have some ports that supply power and some that don't.
- If two lights are on and none is blinking, it's very likely an error.
- If two lights are on and one is blinking, that's normal operation showing that packets are entering and leaving the computer via that interface.
- If two lights are on and one is blinking really fast, that means a lot of packets are moving through the interface.
- The colors are often irrelevant and differ from vendor to vendor. Yellow often signals a gigabit connection, but almost all are, so some vendors have gone back to green. Orange often means 2.5 or 5 gigabits for the connection. 10 gigabit requires a different kind of "plug."

## About ping

A successful ping only means that:

- There is a working network route to the target.
- The target (or something between you and the target) is responding to pings.
- It does not guarantee that the machine is usable, that the operating system is running normally, or that you can log in. A printer, a switch, a router, or even an usuable interface card can answer ping.

A failed ping can mean several things:

- The host is down (no power, no OS, or network interface inactive).
- The host is up but configured to ignore ping.
- A firewall/switch/router in between is dropping ICMP packets.
- There’s no valid route to the target network.


## If you can login...

If you can login to the computer via its console (hey, if you can login via ssh, everything is cool, right?), try either "ip addr" or "ifconfig." They show similar information, organized slightly differently. The commands list all the interfaces on the computer, and you want the one that has the IP address that starts with 141.166. The example is from the workstation in my office. 

ifconfig reports this:
```
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 141.166.181.180  netmask 255.255.252.0  broadcast 141.166.183.255
        inet6 fe80::3eec:efff:fed1:8baf  prefixlen 64  scopeid 0x20<link>
        ether 3c:ec:ef:d1:8b:af  txqueuelen 1000  (Ethernet)
        RX packets 3353538  bytes 1033747837 (985.8 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 482126  bytes 174360994 (166.2 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device memory 0xa5900000-a597ffff
```

and ip addr reports this:

```
3: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 3c:ec:ef:d1:8b:af brd ff:ff:ff:ff:ff:ff
    inet 141.166.181.180/22 brd 141.166.183.255 scope global dynamic noprefixroute eth0
       valid_lft 533301sec preferred_lft 533301sec
    inet6 fe80::3eec:efff:fed1:8baf/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

Both reports show a healthy interface, and together they are the complete picture.

`UP` means the operating system is aware of the hardware, and is running a device driver
of some kind that will allow you to configure this interface and use it.

`RUNNING` means the operating system thinks the interface is currently usable.

`LOWER_UP` means there is an electrical connection to *something* at the other end.

`3c:ec:ef:d1:8b:af` is the MAC address. It is a unique identifier for that one network
interface. There are 281474976710656 (16^12) possible values, so we are not running out.
When a device is added to our network, the folks in IS put the MAC address in their database.
If the MAC address is not in their database, you cannot use it on our network --- unless you are
using the VPN.

## Decoding the IP address.

These two statements convey the same facts:

[1] `inet 141.166.181.180  netmask 255.255.252.0  broadcast 141.166.183.255`

[2] `inet 141.166.181.180/22 brd 141.166.183.255`

The IP address of the workstation is `141.166.181.180`. The netmask of `255.255.252.0` and `/22` both 
give the dimensions of the network that the workstation is a part of: Of the 32 bits that
make up the IP address, the first 22 are the *network address* and the next 10 are the *host address*.
(The base-256 math is left as an exercise for the reader.)
In this example, 

```
141.166.180.0   -- the network address.
141.166.180.1   -- usually the "gateway" to reach other networks.
0  .  0.  1.180 -- the host address on the network.
141.166.183.255 -- the last/largest address on the network.
```

Most of the UR network (everything that starts with 141.166) is cut into /22 chunks, meaning there
are 1024 IP addresses in each chunk.

The rules for network traffic are:

1. Computers on the same network can "see" each other. They send their packets
    directly to each other without needing a router.
1. Computers on different networks require a "route" between their networks to
    communicate.
1. A "router" is a computer with at least two interfaces, and the interfaces are on
    at least two different networks. Internally, its software can move packets from
    one interface to another one. By default, this cannot be done.
1. A "switch" is a computer that allows computers on the same network to talk to each other.
1. Only computers on the same network can determine each other's MAC addresses.









