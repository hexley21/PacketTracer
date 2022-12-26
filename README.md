# PacketTracer

<details open="open">

<summary>Tables of content </summary>

- [PacketTracer](#packettracer)
  - [About](#about)
    - [Before moving on](#before-moving-on)
      - [Modes](#modes)
  - [VLAN](#vlan)
    - [Router on stick](#router-on-stick)
    - [Layer 3 switch](#layer-3-switch)
  - [VLAN](#vlan)
  - [SSH](#ssh)
    - [SSH Configuration (config)](#ssh-configuration-config)
  - [Routing](#routing)
    - [Static Routing](#static-routing)
      - [Summary route calculation](#summary-route-calculation)
    - [EIGRP](#eigrp)
      - [IPV6](#ipv6-eigrp)
      - [Metric](#metric)
    - [OSPF](#ospf)
      - [IPV6](#ipv6-ospfv3)
      - [DR - BDR](#dr---bdr)
    - [RIP](#rip)
  - [NAT](#nat)
    - [Static](#static-nat)
    - [Dynamic](#dynamic-nat)
    - [PAT](#pat)
  - [ACL](#acl)
  - [DHCP](#dhcp)
  - [IPv6](#ipv6)
    - [Configure IPv6](#configure-ipv6)
  - [Port Security](#port-security)
  - [Device commands](#device-commands)
    - [Network commands](#network-commands)
    - [Basic configuration](#basic-configuration)
    - [Switch](#switch)
    - [Router](#router)
    - [Lines (vty/con)](#lines-vtycon)
  - [Subnet Masks](#subnet-masks)

</details>

## About

This is a short summary of commands and knowledge gathered from cisco's netcad courses and from Network Administration books:

[Network Administration 1](https://github.com/hexley21/PacketTracer/blob/e3264c111e7922d33de1a16441ba3edf7715597a/%5BNetwork%20Administration%201%5D%20Routing%20and%20switching%20in%20networks.pdf)

[Network Administration 2](https://github.com/hexley21/PacketTracer/blob/e3264c111e7922d33de1a16441ba3edf7715597a/%5BNetwork%20Administration%202%5D%20Network%20Connections%20and%20WAN%20Technologies.pdf)

### Before moving on

Before moving on, read this paragraph, this will help you to understand this manual better.

#### Modes

Each command has to be launched in specific "scope", in this pagragraph, you will see all these modes and commands to enable them.

__Example:__
> Show all VLANs (exec)
>
> _(means that "show all vlans" command can be launched only from `exec` mode)_
>
> you have to type `enable` in the console

Exec (exec)

    enable

Configuration (config)

    enable
    configure terminal

Interface (config-if)

    enable
    configure terminal
    interface {interface}{port}

> choose interface range, for 1 and more interfaces

    interface range {interface}{port_start} - {port_end}, {interface}{port_start} - {port_end}

Vlan (config-vlan)

    enable
    configure terminal
    vlan {vlan_id}

Console Line (config-con)

    enable
    configure terminal
    line con 0

Console VTY (config-vty)

    enable
    configure terminal
    line vty 0 15

Router EIGRP (eigrp)

    enable
    configure terminal
    router eigrp {process_id}

Ipv6 Router EIGRP (eigrp-ipv6)

    enable
    configure terminal
    ipv6 router eigrp {process_id}

Router OSPF (ospf)

    enable
    configure terminal
    router ospf {process_id}

Ipv6 Router OSPF (ospf-ipv6)

    enable
    configure terminal
    router ospf {process_id}

## VLAN

### Router on stick

1. Create Vlan and assign name (config)

        vlan {number}
        name {vlan-name}

2. Adding interfaces to Vlan

        interface {interface}{port}
        switchport mode access
        switchport access vlan {number}

3. Activating "Trunk Mode" for external interfaces

        interface {ext_interface}{port}
        switchport mode trunk

    > Router - Switch connection interface

4. Create subinterfaces on the router and point them

        interface {interface}{port}.{vlan_num}
        encapsulation dot1Q {vlan_num}
        ip address {ip} {subnet_mask}
        no shut

    > IP of the router to which we will assign this interface

### Layer 3 Switch

1. Create VLANS

        vlan {vlan-num}
        name {vlan-name}

2. Configure Switch Virtual Interfaces on L3 Switch (config)

        interface vlan {vlan-num}
        ip address {ip-address} {mask}

3. Configure Trunking on L3 Switch.

        interface {outside-interface} {port}
        switchport mode trunk
        switchport trunk native vlan {native-vlan-num}

4. Enable routing.

        ip rouing

Add native VLAN to trunk port (config)

    inteface {exit_interface}{port}
    switchport mode trunk
    switchport trunk native vlan {number}

Enable Dynamic trunking protocol {DTP} (config-if)

    switchport mode dynamic {desirable/auto}

Enable no negotiate (config-if)

    switchport nonegotiate

Enable voice in VLAN (config-if)

    mls qos trust cos
    switchport voice vlan {number}

## SSH

Secure Shell (SSH) is a protocol that provides remote Secure (encrypted) managed connection to the device.

### SSH Configuration (config)

1. IP domain configuration

        ip domain-name {your.link}`

2. RSA key pair generation

        crypto key generate rsa

3. User authentication configuration

        username {name} secret {password}

4. VTY lines configuration

        line vty 0 15
        transport input ssh
        login local
        exit

5. Enable SSH ver. 2

        ip ssh version 2
        exit

See SSH configuration (exec)

    show ip ssh

Delete RSA key

    crypto key zeroize rsa

## Routing

|              |RIPv1 |RIPv2 |IGRP  |EIGRP  |OSPF   |IS-IS  |
|--------------|------|------|------|-------|-------|-------|
|Speed Coverage|Slow  |Slow  |Slow  |Fast   |Fast   |Fast   |
|Scalability   |Small |Small |Small |Large  |Large  |Large  |
|Use of VLSM   |No    |Yes   |No    |Yes    |Yes    |Yes    |
|Resource Usage|Low   |Low   |Low   |Medium |High   |High   |
|Implementation|Simple|Simple|Simple|Complex|Complex|Complex|

### Static Routing

    ip route {destination_ip} {destination_subnet} {route - [ip/interface]}

Default route

    ip route 0.0.0.0 0.0.0.0 {route - [ip/interface]}

#### Summary route calculation

Example:

`172.16`.2.0 / 26\
`172.16`.1.128 / 25\
`172.16`.1.0 / 25\
`172.16`.0.0 / 24

all these adresses have `172.16` in common

`172.16.000000`10.00000000\
`172.16.000000`01.10000000\
`172.16.000000`01.00000000\
`172.16.000000`00.00000000

all these adresses share `172.16.000000`, so we get ip summary of:\
`172.16.0.0 / 22`

which gives us a range of `172.16.0.0` to `172.16.3.255`

### EIGRP

> (config)

1. Enable the EIGRP routing process.

        router eigrp {process-num}

2. Assign a router ID.

        eigrp router-id {A.B.C.D}
    > eigrp router-id 1.1.1.1

3. Advertise directly connected networks.

        network {ip_address} {wildcard}
    > add networks to router eigrp config which are connected to router direclty through interface ports

4. Configure passive interfaces. (eigrp)

        passive-interface {interface}{port}`
    > usually you should add ports which are connected to switches or devices

Disable automatic summarization.

    no auto-summary

Propagate the default route in EIGRP. (eigrp)

    redistribute static

Hello interval

    ip hello-interval eigrp {process-num} {interval}

Make every interface passive (eigrp)

    passive-interface default

Display routing protocol parameters

    show ip protocols

EIGRP investigation commands

    show ip eigrp topology
    show ip eigrp interfaces
    show ip eigrp neighbors

#### IPV6-eigrp

> config

1. Enable IPv6 routing. (config)

        ipv6 unicast-routing

2. Enable EIGRP for IPv6 routing.

        ipv6 router eigrp {process-num}
        no shutdown

3. Assign a router ID

        eigrp router-id {A.B.C.D}

4. Configure EIGRP for IPv6 on each interface (config)

        interface {interface} {port}
        ipv6 eigrp {process-num}

5. Configure passive interfaces. (ipv6-eigrp)

        passive-interface {interface}{port}

#### Metric

    router eigrp 1
    metric weights tos
    metric weights 0 0 0 0 0 0

> In case ISP has static routing, write these commands to other routers

    router eigrp {number}`
    redistribute static`

### OSPF

> Router (conf)

1. Configure router IDs. (config)

        route ospf {num}
        router-id {process_id}

2. Configure Networks for OSPF Routing
    > Configure networks using wildcard masks. (config-router)

        network {ip_address} {wildcard_mask} area {number}

    Or Configure OSPF routing on router interfaces. (config)

        interface {interface}{port}
        ip ospf {process-id} area {area-id}

3. Configure Passive Interfaces (config)
        router ospf {num}
        passive-interface {outside-interface}{port}

Propagate the default route in ospf. (ospf)

    default-information originate

Summarization between areas. (ospf)

    area {area-id} range {ip} {mask}

Hello and Dead intervals (config-if)

    ip ospf hello-interval {interval}

    ip ospf dead-interval {interval}

#### IPv6-OSPFv3

> config

1. Enable IPv6 routing. (config)

        ipv6 unicast-routing

2. Enable OSPF for IPv6 routing.

        ipv6 router ospf {process-num}

3. Assign a router ID

        router-id {A.B.C.D}

4. Configure EIGRP for IPv6 on each interface (config)

        interface {interface} {port}
        ipv6 ospf {process-num} area {area}

5. Configure passive interfaces. (ipv6-eigrp)

        passive-interface {interface}{port}

#### DR - BDR

__DR__ - Designated Router\
__BDR__ - Backup Designated Router\
__DROTH__ - Designated Router OTHer\

DR is elected in ospf network by highest priority.\
BDR is elected in ospf network by second highest priority.\
DROTH is given automatically if neither DR nor BDR is obtained\

> To assign DR or BDR to specific routers

1. Assign highest priority to desired router port

        interface {interface1}{port1}
        ip ospf priority {priority-num}

2. Assign second highest priority to desired router port

        interface {interface2}{port2}
        ip ospf priority {priority-num}

3. Clear every neighbor router processes in order to start election (exec)

        clear ip ospf processess

4. Wait and check each router's ospf interface brief and neighbors

        show ip ospf neighbor
        show ip ospf interface brief

Propagate the route in OSPF.

    ip route {ip} {mask} {interface}{port}
    router ospf 1
    default-information originate

Make every interface passive (ospf)

    passive-interface default

Show ip routes learned through OSPF.

    show ip route ospf
    show ip ospf neighbor {interface}{port}
    show ip ospf interface {interface}{port}

### RIP

> Router (conf)
>
> Write these commands to each router and network

    route rip
    version 2
    network {neighbor_router_ip_address}

## NAT

### Static NAT

1. Configure static NAT statements.

    > inside ips are like 192.168.x.x / 172.16.x.x / 10.x.x.x \
    > outside ips can be anything.

        ip nat inside source static {inside-ip} {outside-ip}

2. Configure inside interfaces.
    > interface where your user is connected. Usually g0/0

        interface {interface}{port}
        ip nat inside

3. Configure outside interfaces.

    > interfaces where your router reaches outside networks. Usually s0/0/0

        interface {interface}{port}
        ip nat outside

### Dynamic NAT

1. Configure traffic that will be permitted. (config)

        access-list {acl-num} permit {inside-network-ip} {wildcard-mask}

2. Configure a pool of address for NAT. (config)

        ip nat pool {any_pool_name} {nat-start-ip} {nat-end-ip} netmask {network-mask}

3. Associate ACL with the NAT pool. (config)

        ip nat inside source list {acl-num} pool {your-pool-name}

4. Configure inside interfaces.

        interface {interface}{port}
        ip nat inside

5. Configure outside interfaces.

        interface {interface}{port}
        ip nat outside

### PAT

1. Configure traffic that will be permitted. (config)

        access-list {acl-num} permit {inside-network-ip} {wildcard-mask}

2. Configure a pool of address for NAT. (config)

        ip nat pool {any_pool_name} {nat-start-ip} {nat-end-ip} netmask {network-mask}

3. Associate ACL with the NAT pool and allow addresses to be reused. (config)

        ip nat inside source list {acl-num} pool {your-pool-name} overload

4. Configure inside interfaces.

        interface {interface}{port}
        ip nat inside

5. Configure outside interfaces.

        interface {interface}{port}
        ip nat outside

- __Permit ip addressed other way__ (might be needed)

        ip access-list standard {acl-name}
        permit {network-ip} {wildcard-mask}

- __Step 3 alternative__\
Associate ACL with the NAT interface and allow addresses to be reused.

        ip nat inside source list {acl-num} interface {outside-interface-port} overload

NAT investigation commands. (exec)

    show running-config | include nat
    show ip nat translations
    show ip nat statistics

## ACL

Create standard ACL. (config)

    ip access-list standard {word | 1-99}

Create extended ACL. (config)

    ip access-list extended {word | 100-199}

Add record to standard ACL. (config)

    ip access-list standard {acl-name}
    permit {ip} {wildcard-mask}

Add record to standard ACL other way. (config)

    access-list {word | 1-99} {permit|deny} {ip} {wildcard-mask}

Add record of a host to standard ACL. (config)

    access-list {word | 1-99} {permit|deny} host {ip}

ACL investigation commands. (exec)

    show ip access-lists

## DHCP

> Router (conf)

    ip dhcp pool {name}
    default-router {target_ip}
    network {ip-start-point} {subnet-mask}
    dns-server {dns_address}`

> network is usually like - ___192.168.100.0 255.255.255.0___\
> dns-server is usually like - ___8.8.8.8___

Exclude IPs

    ip dhcp excluded -address {ip-start-point} {ip-end-point}

Server

> Desktop -> IP Configuration: Static \
> and assign all needed IPs
>
> services -> DHCP
>
> pool name: `{name}` \
> default geteway: `{target}` \
> DNS server: `{address}` \
> start IP Address : `ip_start_point` \
> subnet mask: `{subnet}`
> maximum numbers of Users: `{0 - 255}`
>
> __Save__
>
> for each dhcp (if exists), write these commands in router
>
> - `interface {interface} {port}`
> - `ip helper-address {dhcp_server_ip`

## IPv6

### Configure IPv6

1. Enable ipv6 routing (config)

        ipv6 unicast-routing

2. Set ipv6 address

        ipv6 address {address}

3. Set link-local to ipv6

        ipv6 address {address} link-local

## Port Security

Configure port-security(config-if)

Give mac address to port

    switchport port-security mac-address {address/sticky}

> sticky gives mac-addresses dynamicly

Restrict maximum amount of mac addresses given

    switchport port-security maximum {0-15}

Add action on security violation

    switchport port-security violation {shutdown/restrict/protect}

## Device commands

### Network commands

Show all VLANs (exec)

    show vlan brief

Show all interfaces (exec)

    show ip interface brief

Verify ip routes (exec)

    show ip route

Verify ipv6 routes (exec)

    show ipv6 route

### Basic configuration

Show running config (exec)

    show running-config

Save running config to startup config (exec)

    copy running-config startup-config

> Destination filename [startup-config]? [Enter]

Erase startup config (exec)

    erase startup-config
    reload

Show flash memory

    show flash

Delete file

    delete {file}

Show mac address table (exec)

    show mac address-table (dynamic/static/NONE)

Display system hardware and software status (exec)

    show version

Display history of command entered (exec)

    show history

Clear dynamic mac address table (exec)

    clear mac address-table dynamic

Hostname (config)

    hostname {name}

Enable secret (config)

    enable secret {password}

Enable password (config)

    enable password {password}

Enable Password-Encryption (config)

    service password-encryption

Set password min-length (config)

    security passwords min-length

Banner (config)

    banner motd #
    {message}#

> Enter TEXT message.  End with the character #.

Description (config-if)

    description {text}

Restrict unwanted DNS lookup (config)

    no ip domain-lookup

### Switch

### Router

Block line logins after unsuccessful attempts (config)

    login block-for {seconds} attempts {num_of_tries} within {second}

Set clock (config)

    clock set 15:20:00 12 Nov 2020

Enable Duplex (config-if)

> Full-duplex communication increases bandwidth efficiency by allowing both ends of a connection to transmit and receive data simultaneously.

    duplex full
    speed {value}

> val is usually - ___100___

Auto-MDIX (config-if)

> With auto-MDIX enabled, either type of cable can be used to connect to other devices, and the interface automatically adjusts to communicate successfully.

    mdix auto

See mdix state

    show controllers ethernet-controller {interface}{port} phy | include MDIX

### Lines (vty/con)

Disconect after inactivity

    exec-timeout {minutes} {seconds}

Enable Synchronous logging(con):

    logging synchronous

## Subnet Masks

|CIDR|SUBNET MASK    |WILDCARD MASK  |# OF IP ADDRESSES|# OF HOSTS   |
|----|---------------|---------------|-----------------|-------------|
|/32 |255.255.255.255|0.0.0.0        |1                |1            |
|/31 |255.255.255.254|0.0.0.1        |2                |2*           |
|/30 |255.255.255.252|0.0.0.3        |4                |2            |
|/29 |255.255.255.248|0.0.0.7        |8                |6            |
|/28 |255.255.255.240|0.0.0.15       |16               |14           |
|/27 |255.255.255.224|0.0.0.31       |32               |30           |
|/26 |255.255.255.192|0.0.0.63       |64               |62           |
|/25 |255.255.255.128|0.0.0.127      |128              |126          |
|/24 |255.255.255.0  |0.0.0.255      |256              |254          |
|/23 |255.255.254.0  |0.0.1.255      |512              |510          |
|/22 |255.255.252.0  |0.0.3.255      |1,024            |1,022        |
|/21 |255.255.248.0  |0.0.7.255      |2,048            |2,046        |
|/20 |255.255.240.0  |0.0.15.255     |4,096            |4,094        |
|/19 |255.255.224.0  |0.0.31.255     |8,192            |8,190        |
|/18 |255.255.192.0  |0.0.63.255     |16,384           |16,382       |
|/17 |255.255.128.0  |0.0.127.255    |32,768           |32,766       |
|/16 |255.255.0.0    |0.0.255.255    |65,536           |65,534       |
|/15 |255.254.0.0    |0.1.255.255    |131,072          |131,070      |
|/14 |255.252.0.0    |0.3.255.255    |262,144          |262,142      |
|/13 |255.248.0.0    |0.7.255.255    |524,288          |524,286      |
|/12 |255.240.0.0    |0.15.255.255   |1,048,576        |1,048,574    |
|/11 |255.224.0.0    |0.31.255.255   |2,097,152        |2,097,150    |
|/10 |255.192.0.0    |0.63.255.255   |4,194,304        |4,194,302    |
|/9  |255.128.0.0    |0.127.255.255  |8,388,608        |8,388,606    |
|/8  |255.0.0.0      |0.255.255.255  |16,777,216       |16,777,214   |
|/7  |254.0.0.0      |1.255.255.255  |33,554,432       |33,554,430   |
|/6  |252.0.0.0      |3.255.255.255  |67,108,864       |67,108,862   |
|/5  |248.0.0.0      |7.255.255.255  |134,217,728      |134,217,726  |
|/4  |240.0.0.0      |15.255.255.255 |268,435,456      |268,435,454  |
|/3  |224.0.0.0      |31.255.255.255 |536,870,912      |536,870,910  |
|/2  |192.0.0.0      |63.255.255.255 |1,073,741,824    |1,073,741,822|
|/1  |128.0.0.0      |127.255.255.255|2,147,483,648    |2,147,483,646|
|/0  |0.0.0.0        |255.255.255.255|4,294,967,296    |4,294,967,294|
