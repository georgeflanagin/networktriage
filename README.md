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

## Decoding the IP address.

These two statements convey the same fact:

[1] `inet 141.166.181.180  netmask 255.255.252.0  broadcast 141.166.183.255`

[2] `inet 141.166.181.180/22 brd 141.166.183.255`

Both 
