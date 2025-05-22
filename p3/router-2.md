ip link add vxlan10 type vxlan id 10 dstport 4789 local 2.2.2.2 nolearning
ip link add br0 type bridge
ip link set vxlan10 master br0
ip link set eth1 master br0
ip link set vxlan10 up
ip link set br0 up
ip a add 22.10.0.1/24 dev br0

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