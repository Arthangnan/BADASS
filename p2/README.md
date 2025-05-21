## Network configuration
We are going to do configure the /etc/network/interfaces provided by GNS3, which can be configured per node in the `Network configuration` menu.

Routers uses 2 interfaces, `eth1` for the clients and `eth0` for the switch.
|      hostname      | interfaces |     IP      | gateway  |
|--------------------|------------|-------------|----------|
|  host_aattali-1  |    eth1    | 22.10.0.2/24 | 22.10.0.1 |
|  host_aattali-2  |    eth1    | 22.10.0.3/24 | 22.10.0.1 |
| router_aattali-1 |    eth1    | 22.10.0.1/24 |          |
| router_aattali-1 |    eth0    | 99.10.1.1/24 |          |
| router_aattali-2 |    eth1    | 22.10.0.1/24 |          |
| router_aattali-2 |    eth0    | 99.10.1.2/24 |          |