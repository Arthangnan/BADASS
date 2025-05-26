## Host
We are using a basic **alpine** docker image, nothing special there.

## Router
We use **FRRouting** as our routing suite, which bundles the required services : **BGP**, **OSPF**, **IS-IS** and **Zebra**.

The `daemons` config file is used to activate **BGP**, **OSPF** and **IS-IS**, as required. **Zebra** is *pre-activated* by default.

*Vtysh* is the configuration tool (similar to **CISO IOS**) used by **FRRouting**, which we activate.