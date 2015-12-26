# Cisco Cheatsheet
Made by **Mathias Hedberg**
![](https://upload.wikimedia.org/wikipedia/commons/thumb/6/64/Cisco_logo.svg/800px-Cisco_logo.svg.png)

Welcome to the Cisco-Cheatsheet wiki!

Documents covering the basics can be found in the sidebar to the right. Commands are tested with the CISCO Catalyst 2960.

## Getting Started

View the [Basic Configuration w Auth](#1.-Basic-Configuration)

### Disclaimer

This wiki is not guaranteed to be correct, use at your own risk.

# 1. Basic Configuration

## Initial Commands

Commands to bu run right after a fresh boot with no configuration

**Change Hostname:**

    > enable
    # configure terminal
    (config)# hostname [newhostname]
    (config)# exit

**Set IP address & enable interface:**
    
    (config)# interface g0/0
    (config-if)# ip address [ip-address] [subnet]
    (config-if)# no shutdown

**Check interfaces**

    # show ip interface brief
 
**Disable DNS lookup:**

    (config)# no ip domain-lookup

# 2. Authentication

**Enable Secret:**

The user will be prompted to enter this password when entering enable mode

    (config)# enable secret [newsecret]

**Set password for vty & comm-port login:**

VTY:

    > enable
    # configure terminal

Allow 16 different connections (0 to 16)

    (config)# line vty 0 15
    (config-line)# login local

Alternativly one universal password (not reccomended):

    (config-line)# login password [newpass]
    (config-line)# exit

COM-PORT:

    > enable
    # configure terminal
    (config)# line con 0
    (config-line)# login local
    (config-line)# exit

Add the local user:

    (config)# username admin secret [newpasswd]
    (config)# exit

**Save Config:**

    # copy running-config startup-config
    # show startup-config

# 3. Static Routing

##  Configure a static route

![Example](https://raw.githubusercontent.com/metrafonic/Cisco-Cheatsheet/master/images/Screenshot%20from%202015-06-30%2017%3A10%3A41.png)

It is important to remember that in a scenario where you may not need to add a static route on your gateway router to access a different router, you will most likely need to configure a route back to the client. The remote router will need a route to your internal subdomain via your access router.

**Recursive**

    (config)# ip route [nettID] [subnet] [router ip to use]

**Directly Connected**
    
    (config)# ip route [ip-address] [subnet] [interface]

**Set a default route**

    (config)# ip route 0.0.0.0 0.0.0.0 [gateway router]

**Check route**

    # show ip route

# 4. EIGRP & OSPF Routing

## EIGRP:
###  EIGRP Usage
![Example](https://raw.githubusercontent.com/metrafonic/Cisco-Cheatsheet/master/images/Screenshot%20from%202015-06-30%2017%3A59%3A23.png)

[**Download EIGRP Example**](https://github.com/metrafonic/Cisco-Cheatsheet/blob/master/Packet%20Tracer/EIGRP.pkt)

In the example above, we have four routers connected together via switches, with two end devices added to the mix. To make the job of setting up routing on this network much easier, we use EIGRP. Router 10 would in this example, advertise its neighbours to the rest of the network, so that a topology can be built. These adjacent networks to be advertised must be set manually, otherwise it is almost completely automatic.

**Enable EIGRP**

    # configure teminal
    (config)# router eigrp [autonomous-system-number]
    (config-router)# router-id [ipv4-address]  //Not required

**Enable for adjacent subnets**

Replace nettID with the subnets the router is directly connected to, this will often be more than one. Default is a /16 subnet when no subnet is given:

    (config-router)# network [nettID] [wildcard mask]

To find the inverted subnet, calculate 255.255.255.255 minus your given subnet

**Example:**

In the example above, router 10 would have the following:

    # configure teminal
    (config)# router eigrp 10
    (config-router)# network 192.168.1.0 0.0.0.255
    (config-router)# network 192.168.2.0 0.0.0.255

As neighbours are added, a prompt will display on screen alerting the user of the neighbours

### Verify EIGRP Routing

**View Neighbors**

    # show ip eigrp neighbors

**View ip EIGRP routing table**

    # show ip route eigrp

**View EIGRP topology table**

    # show ip eigrp topology

**Verify EIGRP routing parameters**

    # show ip protocols

### Modifying Bandwidth

Useful for checking the functionality of your setup.

    # interface [interface]
    # bandwidth [Kbit/sec]

### Configure passive interfaces

Useful to use on the interface to be used on the client side, to avoid MITM attacks.

    # router eigrp [used AS number]
    (config-router)# passive-interface [interface]
### Remove EIGRP

    (config)# no router eigrp [autonomous-system-number]

## OSPF:

Guide can be found here:
https://www.petri.com/how-to-configure-ospf-in-cisco-ios

# 5. DHCP Configuration

### IP relaying of DHCP (IPHELPER)

![Example](https://raw.githubusercontent.com/metrafonic/Cisco-Cheatsheet/master/images/Screenshot%20from%202015-06-30%2020%3A07%3A32.png)

**Challenge** I have not managed to figure out how this can be done to request an ip over multiple routers. As of right now i am only able to do this on a router that is connected to both subnets directly. In the topology above, we have one DHCP server in the middle, three relays and three clients, though only two of the relay servers work correctly.

**Enable IPHELPER on router**

    # conf t
    (config)# int [interface to relay to]
    (config-if)# ip helper [dhcp server address]

**Check IPHELPER configuration**

    # show ip int [interface]

### DHCP Server Configuration

**Creating the pool**

    (config)# ip dhcp pool [nameofpool]
    (dhcp-config)# network [nettID] [subnet]

Note that the nettID where the client request is coming from will be checked against the nettID and subnet set here. if the two do not match, an address will not be allocated.

    (dhcp-config)# default-router [dafault router]
    (dhcp-config)# dns-server [dafault dns]
    (dhcp-config)# domain-name [example.com]
    (dhcp-config)# lease [days]
    (dhcp-config)# exit

**Excluding Addresses**

    (config)# ip dhcp exluded-address [start reserved ip] [end reseved ip]

### Debugging

**View DHCP address leases**

    # show ip dhcp binding

**View DHCP pool statistics**

    # show ip dhcp server statistics

**View DHCP pool settings**

    # show ip dhcp pool

# 6. IPV6 Configuration

### Enabling IPv6
**Initialize IPv6**

    # conf t
    (config)# ipv6 unicast-routing

**Configure an andress on an interface**

    (config)# interface [interface]
    (config-if)# ipv6 address [ipv6 prefix]/[prefix length] eui-64
    (config-if)# ipv6 address [link-local] link-local //not required
    (config-if)# no shutdown
    (config-if)# exit

Note, the ipv6 prefix will look like a netid will usually end in 0, the wui-64 command allocates the unique portion of the computer to the address. This in a whole will be the same as witing  
**Example:**

    (config-if)# ipv6 address 2001:DB8:ACAD:A::/64 eui-64

### IPv6 Routing:
**Static Route**

    (config)# ipv6 route [ipv6 prefix]/[prefix length] [next hop ipv6 address]

**Default IPv6 static route**

    (config)# ipv6 route ::/0 [interface]

**EIGRP Dynamic Routing**

Enable ipv6 routing:

    (config)# ipv6 unicast-routing

Start eigpr protocol

    (config)# ipv6 router eigrp [AS-number]

Configure EIGRP for a 32-bit address for the router ID.

    (config-rtr)# eigrp router-id [32 bit ip id] //feks 1.1.1.1

Enable:

    (configure-rtr)# no shutdown

# 7. NAT Protocol Mapping

In progress

# 8. Password Recovery 

In progress

# 9. Telnet & SSH Configuration

### Adding telnet access

Configure an admin user and password with level 15 access:

    (config)# username admin priv 15 secret cisco

Enable telnet access for a variable number of sessions:

    (config)# line vty 0 [number of sessions]
    (config-line)# login local
    (config-line)# end

### Enable SSH

Set a domain name:

    ip domain-name [domain.no]

Generate keys:

    R1(config)# crypto key gen rsa

Set time if time sync error appears:

    #clock set 13:00:00 2 JUL 2015

Update ssh version:

    (config)# ip ssh version 2

Connect from client.

# 10. Configuration Backup & Restore

Backup to tftp

    #copy running-config tftp:

Restore from tftp:

    #copy tftp: running-config
