# Setup a gateway with no firewall

In order to easily allow both my home lab and home routers to both connect to the internet I am using my CCR226 to act as a gateway.  This is behind my ISP provided router and has no firewall support.

This configures the switch to have two bridges, `bridge1` and `bridge3`.  `bridge1` bridges `ether1` through `ether8`.  `ether1` is configured to have `192.168.88.2` as the address.  `bridge3` bridges `ether18` through `ether24`, along with `sfp-sfpplus1` and `sfpplus2`.  `ether17` is setup as the WAN and uses DHCP to get it's address.  A DHCP server is setup on `bridge3` and assigns `192.168.89.0/24` addresses.

## Instructions

These steps assume the router was just reset and has the default configuration allowing access on the `192.168.88.1` address with no password.  You should also be directly connected to `ether1` and have a static IP in the `192.168.88.0/24` network.

First you will need to remove the `ether1` interface from the default bridge with this command:

> [!WARNING] 
> This will cause you to lose connectivity.

    /interface bridge port remove [find interface=ether1]

You should be able to connect to the switch using the neighborhood tab in winbox.

Next run this command to change the default address and to move it to the `ether1` interface.

    /ip address set [find address="192.168.88.1/24"] address=192.168.88.2/24 interface=ether1 comment=""

Next, because of my OCD, remove the default bridge:

    /interface bridge port remove [find bridge=bridge]
    /interface bridge remove [find bridge=bridge]

To setup `bridge1` run the following commands:

    /interface bridge add name=bridge1
    /interface bridge port add bridge=bridge1 interface=ether1
    /interface bridge port add bridge=bridge1 interface=ether2
    /interface bridge port add bridge=bridge1 interface=ether3
    /interface bridge port add bridge=bridge1 interface=ether4
    /interface bridge port add bridge=bridge1 interface=ether5
    /interface bridge port add bridge=bridge1 interface=ether6
    /interface bridge port add bridge=bridge1 interface=ether7
    /interface bridge port add bridge=bridge1 interface=ether8

The following commands will setup `bridge3`:

    /interface bridge add name=bridge3
    /interface bridge port add bridge=bridge3 interface=ether18
    /interface bridge port add bridge=bridge3 interface=ether19
    /interface bridge port add bridge=bridge3 interface=ether20
    /interface bridge port add bridge=bridge3 interface=ether21
    /interface bridge port add bridge=bridge3 interface=ether22
    /interface bridge port add bridge=bridge3 interface=ether23
    /interface bridge port add bridge=bridge3 interface=ether24
    /interface bridge port add bridge=bridge3 interface=sfp-sfpplus1
    /interface bridge port add bridge=bridge3 interface=sfpplus2

For ease of management, setup some lists with these commands:

    /interface list add name=WAN
    /interface list add name=LAN
    /interface list member add interface=ether17 list=WAN
    /interface list member add interface=bridge3 list=LAN

Next we will setup NAT on the WAN:

    /ip firewall nat add action=masquerade chain=srcnat out-interface-list=WAN

Then we need to configure the addresses for the WAN and LAN:

    /ip dhcp-client add disabled=no interface=ether17
    /ip address add address=192.168.89.1/24 interface=bridge3 network=192.168.89.0

Finally we will setup the DHCP server:

    /ip pool add name=ip-pool-bridge3 ranges=192.168.89.100-192.168.89.200
    /ip dhcp-server add address-pool=ip-pool-bridge3 disabled=no interface=bridge3 name=dhcp-server-bridge3
    /ip dhcp-server network add address=192.168.89.0/24 dns-server=8.8.8.8 gateway=192.168.89.1
