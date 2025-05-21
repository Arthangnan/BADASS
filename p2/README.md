## Network configuration
We are going to do configure the `/etc/network/interfaces` provided by **GNS3**, which can be configured per node in the `Network configuration` menu.

Routers uses 2 interfaces, `eth1` for the clients and `eth0` for the switch.
The `eth1` interface will not be configured with **GNS3**, since it doesn't have any ip address.
|      hostname      | interfaces |     IP      | gateway  |
|--------------------|------------|-------------|----------|
|  host_aattali-1  |    eth1    | 22.10.0.2/24 | 22.10.0.1 |
|  host_aattali-2  |    eth1    | 22.10.0.3/24 | 22.10.0.1 |
| router_aattali-1 |    eth1    |  |          |
| router_aattali-1 |    eth0    | 99.10.1.1/24 |          |
| router_aattali-2 |    eth1    |  |          |
| router_aattali-2 |    eth0    | 99.10.1.2/24 |          |

## Unicast with static flooding
Since we are not using **multicast**, we need to configure every **VTEP**s so they can be included in the loop.
We need to handle **BUM** frames *(broadcast, unknown-unicast and multicast)* which are sent when the sender doesn't know the destination **MAC** address.

The source **VTEP** will create as many other **VTEP**s copies of the encapsulated **BUM** frame (which is 1 for us) and send a separate unicast **IP** packet to the **VTEP**.
The **VXLAN** will still learn the remote addresses using **source-address learning**, mapping the **MAC** addresses as needed in the **FDB**.
Thus, the only duplicated traffic is the **BUM** frames, the rest of the traffic will reach only the revelant **VTEP**.

Everything will happens on both routers, adapt the configuration as needed, we will be using `router-1` as an example.
### VXLAN and bridge creation
#### VXLAN
```sh
ip link add vxlan10 type vxlan id 10 dstport 4789 local 99.10.1.1
```
We use the **VNI** `10` as required in the subject. The destination port for the UDP packets is the **IANA**-assigned one. The local address is the source address on the `eth0` interface.
#### Bridge
```sh
ip link add br0 type bridge
ip link set vxlan10 master br0
ip link set eth1 master br0
```
We create a bridge called `br0`, we add both the local host interface `eth1` and the **VXLAN** tunnel interface `vxlan10` to the **bridge**.
### Interfaces and bridge configuration
#### Interfaces activation
We activate both the **bridge** and the **VXLAN**.
```sh
ip link set br0 up
ip link set vxlan10 up
```
#### FDB tables
We activate the **static flooding** method by adding in the **FDB tables** of the bridge every **VEP**s for our **BUM** traffic.
We only have one other **VEP** in this setup.
```sh
bridge fdb append 00:00:00:00:00:00 dev vxlan10 dst 99.10.1.2
```
``00:00:00:00:00:00`` is a *special* ***MAC*** *address* representing the **BUM** traffic.
#### IP assignment for the bridge
Since we added `eth1` to the **bridge**, all the traffic coming from the hosts on `eth1` will be picked up.
We want to ping our **routers**, so we need to assign the router ip to the `br0` interface and not on the `eth1` interface.
```sh
ip a add 22.10.0.1/24 dev br0
```

## Multicast
todo