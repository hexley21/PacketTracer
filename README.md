# PacketTracer

<details open="open">

<summary>Tables of content </summary>

- [About](#about)
  - [Modes](#modes)
- [VLAN](#vlan)
- [SSH](#ssh)
- [Routing](#routing)
  - [Static](#static-routing)
    - [Summary route](#summary-route-calculation)
- [Device commands](#device-commands)
  - [Network commands](#network-commands)
  - [Basic configuration](#basic-configuration)
- [Subnet Masks](#subnet-masks)

</details>

## About

This is a short summary of commands and knowledge gathered from cisco's netcad courses and from Network Administration books:

[Network Administration 1](https://github.com/hexley21/PacketTracer/blob/e3264c111e7922d33de1a16441ba3edf7715597a/%5BNetwork%20Administration%201%5D%20Routing%20and%20switching%20in%20networks.pdf)

[Network Administration 2](https://github.com/hexley21/PacketTracer/blob/e3264c111e7922d33de1a16441ba3edf7715597a/%5BNetwork%20Administration%202%5D%20Network%20Connections%20and%20WAN%20Technologies.pdf)

### Before moving on

It is recommended to read through everything in this paragraph, before moving on, to get rid of confusions

#### Modes

Here is a list of modes and entering guide, in which you have to launch your specific command

>Example: Show all VLANs (exec)
>
>menas that you should launch given command from specific mode
>
> you have to type `enable` at CLI main page

##### Exec (exec)

- `enable`

##### Configuration (config)

- `enable`
- `configure terminal`

##### Interface (config-if)

- `enable`
- `configure terminal`
- `interface {interface}{port}`

    > choose interface range, for 1 and more interfaces

- `interface range {interface}{port_start} - {port_end}, {interface}{port_start} - {port_end}`

##### Vlan (config-vlan)

- `enable`
- `configure terminal`
- `vlan {vlan_id}`

##### Console Line (config-con)

- `enable`
- `configure terminal`
- `line con 0`

> synchronous logging:

- `logging synchronous`

##### Console VTY (config-vty)

- `enable`
- `configure terminal`
- `line vty 0 15`

## VLAN

1. Create Vlan and assign name (config)
    - `vlan {number}`
    - `name {vlan-name}`

2. Adding interfaces to Vlan
    - `interface {interface}{port}`
    - `switchport mode access`
    - `switchport access vlan {number}`

3. Activating "Trunk Mode" for external interfaces
    - `interface {ext_interface}{port}`
    - `switchport mode trunk`
    > Router - Switch connection interface
4. Create subinterfaces on the router and point them
    - `interface {interface}{port}.{vlan_num}`
    - `encapsulation dot1Q {vlan_num}`
    - `ip address {ip} {subnet_mask}`
    > IP of the router to which we will assign this interface
    - `no shut`

## SSH

Secure Shell (SSH) is a protocol that provides remote Secure (encrypted) managed connection to the device.

### SSH Configuration (config)

1. IP domain configuration
    - `ip domain-name {your.link}`
2. RSA key pair generation
    - `crypto key generate rsa`
3. User authentication configuration
    - `username {name} secret {password}`
4. VTY lines configuration
    - `line vty 0 15`
    - `transport input ssh`
    - `login local`
    - `exit`
5. Enable SSH ver. 2
    - `ip ssh version 2`
    - `exit`

#### See SSH configuration (exec)

- `show ip ssh`

#### Delete RSA key

- `crypto key zeroize rsa`

## Routing

|              |RIPv1 |RIPv2 |IGRP  |EIGRP  |OSPF   |IS-IS  |
|--------------|------|------|------|-------|-------|-------|
|Speed Coverage|Slow  |Slow  |Slow  |Fast   |Fast   |Fast   |
|Scalability   |Small |Small |Small |Large  |Large  |Large  |
|Use of VLSM   |No    |Yes   |No    |Yes    |Yes    |Yes    |
|Resource Usage|Low   |Low   |Low   |Medium |High   |High   |
|Implementation|Simple|Simple|Simple|Complex|Complex|Complex|

### Static Routing

- `ip route {destination_ip} {destination_subnet} {route - [ip/interface]}`

    Default route

- `ip route 0.0.0.0 0.0.0.0 {route - [ip/interface]}`

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

## Device commands

### Network commands

#### Show all VLANs (exec)

- `show vlan brief`

#### Show all interfaces (exec)

- `show ip interface brief`

#### Verify ip routes (exec)

- `show ip route`

    or

- `show ipv6 route`

### Basic configuration

#### Show running config (exec)

- `show running-config`

#### Save running config (exec)

- `copy running-config startup-config`

> Destination filename [startup-config]? [Enter]

#### Erase startup config (exec)

- `erase startup-config`
- `reload`

#### Show flash memory

- `show flash`

#### Delete file

- `delete {file}`

#### Show mac address table (exec)

- `show mac address-table (dynamic/static/NONE)`

#### Display system hardware and software status (exec)

- `show version`

#### Display history of command entered (exec)

- `show history`

#### Clear dynamic mac address table (exec)

- `clear mac address-table dynamic`

#### Hostname (config)

- `hostname {name}`

#### Enable secret (config)

- `enable secret {password}`
    > Prefered way

#### Enable password (config)

- `enable password {password}`

#### Enable Password-Encryption (config)

- `service password-encryption`

#### Set password min-length (config)

- `security passwords min-length`

#### Banner (config)

- `banner motd #`

    > Enter TEXT message.  End with the character #.

- `{message}#`

#### Description (config-if)

- `description {text}`

#### Restrict unwanted DNS lookup (config)

- `no ip domain-lookup`

### Switch

### Router

#### Block line logins after unsuccessful attempts (config)

- `login block-for {seconds} attempts {num_of_tries} within {second}`

#### Set clock (config)

- `clock set 15:20:00 12 Nov 2020`

#### Enable ipv6 routing (config)

- `ipv6 unicast-routing`

#### Enable Duplex (config-if)

> Full-duplex communication increases bandwidth efficiency by allowing both ends of a connection to transmit and receive data simultaneously.

- `duplex full`
- `speed {val}` (usually: 100)

#### Auto-MDIX (config-if)

> With auto-MDIX enabled, either type of cable can be used to connect to other devices, and the interface automatically adjusts to communicate successfully.

- `mdix auto`

See mdix state

- `show controllers ethernet-controller {interface}{port} phy | include MDIX`

### Lines (vty/con)

#### Disconect after inactivity

- `exec-timeout {minutes} {seconds}`

## Subnet Masks

|CIDR|SUBNET MASK    |WILDCARD MASK  |# OF IP ADDRESSES|# OF USABLE IP ADDRESSES|
|----|---------------|---------------|-----------------|------------------------|
|/32 |255.255.255.255|0.0.0.0        |1                |1                       |
|/31 |255.255.255.254|0.0.0.1        |2                |2*                      |
|/30 |255.255.255.252|0.0.0.3        |4                |2                       |
|/29 |255.255.255.248|0.0.0.7        |8                |6                       |
|/28 |255.255.255.240|0.0.0.15       |16               |14                      |
|/27 |255.255.255.224|0.0.0.31       |32               |30                      |
|/26 |255.255.255.192|0.0.0.63       |64               |62                      |
|/25 |255.255.255.128|0.0.0.127      |128              |126                     |
|/24 |255.255.255.0  |0.0.0.255      |256              |254                     |
|/23 |255.255.254.0  |0.0.1.255      |512              |510                     |
|/22 |255.255.252.0  |0.0.3.255      |1,024            |1,022                   |
|/21 |255.255.248.0  |0.0.7.255      |2,048            |2,046                   |
|/20 |255.255.240.0  |0.0.15.255     |4,096            |4,094                   |
|/19 |255.255.224.0  |0.0.31.255     |8,192            |8,190                   |
|/18 |255.255.192.0  |0.0.63.255     |16,384           |16,382                  |
|/17 |255.255.128.0  |0.0.127.255    |32,768           |32,766                  |
|/16 |255.255.0.0    |0.0.255.255    |65,536           |65,534                  |
|/15 |255.254.0.0    |0.1.255.255    |131,072          |131,070                 |
|/14 |255.252.0.0    |0.3.255.255    |262,144          |262,142                 |
|/13 |255.248.0.0    |0.7.255.255    |524,288          |524,286                 |
|/12 |255.240.0.0    |0.15.255.255   |1,048,576        |1,048,574               |
|/11 |255.224.0.0    |0.31.255.255   |2,097,152        |2,097,150               |
|/10 |255.192.0.0    |0.63.255.255   |4,194,304        |4,194,302               |
|/9  |255.128.0.0    |0.127.255.255  |8,388,608        |8,388,606               |
|/8  |255.0.0.0      |0.255.255.255  |16,777,216       |16,777,214              |
|/7  |254.0.0.0      |1.255.255.255  |33,554,432       |33,554,430              |
|/6  |252.0.0.0      |3.255.255.255  |67,108,864       |67,108,862              |
|/5  |248.0.0.0      |7.255.255.255  |134,217,728      |134,217,726             |
|/4  |240.0.0.0      |15.255.255.255 |268,435,456      |268,435,454             |
|/3  |224.0.0.0      |31.255.255.255 |536,870,912      |536,870,910             |
|/2  |192.0.0.0      |63.255.255.255 |1,073,741,824    |1,073,741,822           |
|/1  |128.0.0.0      |127.255.255.255|2,147,483,648    |2,147,483,646           |
|/0  |0.0.0.0        |255.255.255.255|4,294,967,296    |4,294,967,294           |
