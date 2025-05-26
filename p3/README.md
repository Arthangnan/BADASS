## Network configuration
We are going to do configure the `/etc/network/interfaces` provided by **GNS3**, which can be configured per node in the `Network configuration` menu.

**Leafs** uses *2 interfaces*, one for communicating with their host, and other one for the **route reflector**.
The **RR** has *3 interfaces* connected to the 3 **leafs**.
Only the intrface for the hosts will be configured with **GNS3**, the configuration for the routers will be done via **FRRouting**.
|      hostname   | interfaces  |     IP       | gateway   |
|-----------------|-------------|--------------|-----------|
| host-aattali-1  |    eth1     | 22.10.0.2/24 | 22.10.0.1 |
| host-aattali-2  |    eth0     | 22.10.0.3/24 | 22.10.0.1 |
| host-aattali-2  |    eth0     | 22.10.0.4/24 | 22.10.0.1 |
| r-aattali-1     |    eth0     | 10.10.0.1/29 |           |
| r-aattali-1     |    eth1     | 10.10.0.2/29 |           |
| r-aattali-1     |    eth2     | 10.10.0.3/29 |           |
| r-aattali-1     |    lo       | 1.1.1.1/29   |           |
| r-aattali-2     |    eth0     | 10.10.0.4/29 |           |
| r-aattali-2     |    eth1     | 22.10.0.1/24 |           |
| r-aattali-2     |    lo       | 1.1.1.2/29   |           |
| r-aattali-3     |    eth0     | 22.10.0.1/24 |           |
| r-aattali-3     |    eth1     | 10.10.0.5/29 |           |
| r-aattali-3     |    lo       | 1.1.1.3/29   |           |
| r-aattali-4     |    eth0     | 22.10.0.1/24 |           |
| r-aattali-4     |    eth1     | 10.10.0.5/29 |           |
| r-aattali-4     |    lo       | 1.1.1.4/29   |           |

> [!CAUTION]
> When operating inside the **routers**, switch to `bash` to use a compliant `iptools2` cli.

## Route reflector
Our goal is to establish a **BGP peer** in **EVPN mode**, which will serves as a **route reflector**, permitting us to avoid a *full-mesh layout* between each **routers**. The **underlay** routing will be done with **OSPF**, using *loopback addresses* in the `1.1.1.0/29` range.
### FRRouting configuration
#### Interfaces
We disable **IPv6 forwarding** since we are forced to only use **OSPFv2**, which doesn't have **IPv6** support.
```sh
zebra
    no ipv6 forwarding
```
Next, we are going to add **ip addresses** for every **ethernet interfaces** per our **topology**, and our **loopback address** with a *restricted CIDR range* since the **routing** will be managed by **OSPF**.
```sh
interface eth0
    ip address 10.10.0.1/29
!
interface eth1
    ip address 10.10.0.2/29
!
interface eth2
    ip address 10.10.0.3/29
!
interface lo
    ip address 1.1.1.1/32
!
```
#### BGP EVPN
The meat of the configuration is here. We use the **AS** number 8, and indicate we are not going to only use IPv4 as routing information (since we are using **BGP** to route **MAC addresses** with **EVPN**).
```sh
router bgp 8
    no bgp default ipv4-unicast
```
Since we have **3 peers** (our *leaves*), it is useful to create a **peer-group**. Our leaves are in the same **AS**, and are *reachable* in our **loopback interface** managed by **OSPF**. The `extended-nexthop` **capability** is needed for **EVPN**. We also need specify the **CIDR ranges** of our **peer-group**.
```sh
    neighbor leafs peer-group
    neighbor leafs remote-as 8
    neighbor leafs capability extended-nexthop
    neighbor leafs update-source lo
    bgp listen range 1.1.1.0/29 peer-group leafs
```
We then activate **EVPN** for our **peer-group**, and advertise our capability as a **route reflector**.
```sh
    address-family l2vpn evpn
        neighbor leafs activate
        neighbor leafs route-reflector-client
    exit-address-family
    !
```
#### OSPF
We let **OSPF** advertise and manage our **interfaces**, in the default `area 0`.
```sh
router ospf
    network 0.0.0.0/0 area 0
```

## Leafs
### VXLAN
For our **VXLAN** configuration, we advertise our local **loopback address** which is managed by **OSPF**. The `nolearning` flag is important, since we are using **BGP EVPN** to learn the **MAC addresses** of our **VTEP**s. The rest was already explained before.
```sh
ip link add vxlan10 type vxlan id 10 dstport 4789 local 1.1.1.2 nolearning
ip link add br0 type bridge
ip link set vxlan10 master br0
ip link set eth1 master br0
ip link set vxlan10 up
ip link set br0 up
ip a add 22.10.0.1/24 dev br0
```
### FRRouting configuration
#### Interfaces
The configuration is similar to our **RR**, but we specify which **interface** will be managed by **OSPF**, since our interface linked to our host need to be **outside** this **routing scheme**.
```sh
zebra
    no ipv6 forwarding
!
interface eth0
    ip address 10.10.0.4/29
    ip ospf area 0
!
interface lo
    ip address 1.1.1.2/32
    ip ospf area 0
!
```
#### BGP EVPN
We have only one **peer**, which is our **RR** at `1.1.1.1`, so we are not going to use a **peer-groupe there**. We specify to the **EVPN** module that we want to advertise all of our **VNI**s (even if we only have one).
```sh
router bgp 8
    no bgp default ipv4-unicast
    neighbor 1.1.1.1 remote-as 8
    neighbor 1.1.1.1 capability extended-nexthop
    neighbor 1.1.1.1 update-source lo
    !
    address-family l2vpn evpn
        neighbor 1.1.1.1 activate
        advertise-all-vni
    exit-address-family
    !
!
```
#### OSPF
**OSPF** routing need to be **activated**, even if we specified our interfaces before.
```sh
router ospf
```