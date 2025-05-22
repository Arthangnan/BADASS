todo

subnet vxlan 22.10.0.x/24
router : 22.10.0.1
clients etc

subnet underlay 10.10.0.x/29
router1 eth0 : 10.10.0.1
router1 eth1 : 10.10.0.2

subnet loopback
router1 : 1.1.1.1
router2 : 2.2.2.2
router3 : 3.3.3.3
router4 : 4.4.4.4

--------

# config reflector
## config frrouting (via vtysh)
!
zebra
    no ipv6 forwarding
!
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
router bgp 8
    no bgp default ipv4-unicast
    neighbor leafs peer-group
    neighbor leafs remote-as 8
    neighbor leafs capability extended-nexthop
    neighbor leafs update-source lo
    neighbor 2.2.2.2 peer-group leafs
    neighbor 3.3.3.3 peer-group leafs
    neighbor 4.4.4.4 peer-group leafs
    !
    address-family l2vpn evpn
        neighbor leafs activate
        neighbor leafs route-reflector-client
    exit-address-family
    !
!
router ospf
    network 0.0.0.0/0 area 0
!

# config router
## vxlan/bash
ip link add vxlan10 type vxlan id 10 dstport 4789 local 2.2.2.2 nolearning
ip link add br0 type bridge
ip link set vxlan10 master br0
ip link set eth1 master br0
ip link set vxlan10 up
ip link set br0 up
ip a add 22.10.0.1/24 dev br0

## config ffrouting (via vtysh)
!
zebra
    no ipv6 forwarding
!
interface eth0
    ip address 10.10.0.4/29
    ip ospf area 0
!
interface lo
    ip address 2.2.2.2/32
    ip ospf area 0
!
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
router ospf
!

---
ip link add vxlan10 type vxlan id 10 dstport 4789 local 3.3.3.3 nolearning
ip link add br0 type bridge
ip link set vxlan10 master br0
ip link set eth0 master br0
ip link set vxlan10 up
ip link set br0 up
ip a add 22.10.0.1/24 dev br0

## config ffrouting (via vtysh)
!
zebra
    no ipv6 forwarding
!
interface eth1
    ip address 10.10.0.5/29
    ip ospf area 0
!
interface lo
    ip address 3.3.3.3/32
    ip ospf area 0
!
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
router ospf
!

---
ip link add vxlan10 type vxlan id 10 dstport 4789 local 4.4.4.4 nolearning
ip link add br0 type bridge
ip link set vxlan10 master br0
ip link set eth0 master br0
ip link set vxlan10 up
ip link set br0 up
ip a add 22.10.0.1/24 dev br0

## config ffrouting (via vtysh)
!
zebra
    no ipv6 forwarding
!
interface eth2
    ip address 10.10.0.6/29
    ip ospf area 0
!
interface lo
    ip address 4.4.4.4/32
    ip ospf area 0
!
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
router ospf
!