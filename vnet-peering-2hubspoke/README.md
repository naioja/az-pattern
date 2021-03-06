<properties
pageTitle= 'how to create two Azure hub-spoke VNets interconnected by global VNet peering by ARM'
description= "ARM template to create two Azure hub-spoke VNets interconnected by global VNet peering"
documentationcenter: na
services=""
documentationCenter="na"
authors="fabferri"
manager=""
editor=""/>

<tags
   ms.service="howto-example-Azure"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="26/07/2018"
   ms.review="23/03/2020"
   ms.author="fabferri" />

# How to create two hub-spoke VNets interconnected by VNet peering


The article describes a scenario with Azure VNet in peering. The network diagram is reported below:

[![1]][1]

the configuration aim to make intercommunication between VNetSpoke1 and vNetSpoke2 with traffic in transit through the VMs nva1 and nva2. The VMs nva1 and nva2 are two CentOS VMs,  with ip forwarding enabled. A Linux VMs with ip forwarding do not make traffic inspection like security Network Virtual appliances (NVAs)  but it is pretty useful to  check the routing end-to-end and the consistency of UDRs (User Defined Routes).
Below a network diagram zoon-in, with UDR and commands to generate traffic between two spoke VNets.

[![2]][2]

The ARM template creates the full environment, inclusive of UDRs. After the deployment, run the step reported below to activate the ip forwarding in nva1 and nva2.

> [!NOTE]
> Before spinning up the ARM template you should:
> * set the Azure subscription name in the file **hub-spoke.ps1**
> * set the administrator username and password in the file **hub-spoke.json**
>


## <a name="EnableIPForwarding"></a>1. Enable ip forwarding on nva1 and nva2

Enable ip forwarding in nva1 and nva2:

```
sed -i -e '$a\net.ipv4.ip_forward = 1' /etc/sysctl.conf
systemctl restart network.service
sysctl net.ipv4.ip_forward
```

### <a name="Effective routes-spoke1"></a>2. Effective routes

#### vm-spoke1
|Source |	State   |	Address Prefixes|	Next Hop Type    |	Next Hop Type IP Address|	User Defined Route Name|
| ----- |:-------:|:----------------|:----------------:|:-----------------------:|:----------------------:|
|Default|	Active  |	10.0.3.0/24     |	Virtual network  |	-|	-|
|Default|	Active	|0.0.0.0/0        |	Internet         |	-|	-|
|Default|	Active	|10.0.0.0/8       |	None	           |-|	-|
|Default|	Active	|100.64.0.0/10    |	None	           |-|	-|
|Default|	Active	|192.168.0.0/16   |	None	           |-|	-|
|User   |	Active	|10.0.2.0/24      |	Virtual appliance|	10.0.1.10|	route-to-hub2|
|User   |	Active	|10.0.4.0/24      |	Virtual appliance|	10.0.1.10|	route-to-spoke2|
|User   |	Active	|10.0.1.0/24      |	Virtual appliance|	10.0.1.10|	route-to-hub1|
|Default|	Invalid	|10.0.1.0/24      |	VNetGlobalPeering|	-|	-|


#### vm-spoke2
|Source|	State	  |Address Prefixes|	Next Hop Type    |Next Hop Type IP Address |	User Defined Route Name|
| ----- |:-------:|:---------------|:-----------------:|:-----------------------:|:----------------------:|
|Default|	Active  |	10.0.4.0/24    |	Virtual network  |	-|	-|
|Default|	Active  |	0.0.0.0/0      |	Internet         |	-|	-|
|Default|	Active  |	10.0.0.0/8     |	None             |	-|	-|
|Default|	Active  |	100.64.0.0/10  |	None             |	-|	-|
|Default|	Active  |	192.168.0.0/16 |	None             |	-|	-|
|User|	Active    |	10.0.1.0/24    |	Virtual appliance|	10.0.2.10|	route-to-hub1|
|User|	Active    |	10.0.3.0/24    |	Virtual appliance|	10.0.2.10|	route-to-spoke1|
|User|	Active    |	10.0.2.0/24    |	Virtual appliance|	10.0.2.10|	route-to-hub2|
|Default|	Invalid |	10.0.2.0/24    |	VNetGlobalPeering|	-|	-|


#### vm-hub1-subnet2
|Source|	State	  |Address Prefixes|	Next Hop Type      |Next Hop Type IP Address |	User Defined Route Name|
| ----- |:-------:|:---------------|:-----------------:|:-----------------------:|:----------------------:|
|Default|	Active	|10.0.1.0/24	   | Virtual network	 |-|	-|
|Default|	Active	|0.0.0.0/0	     | Internet	         |-|	-|
|Default|	Active	|10.0.0.0/8	     | None	             |-|	-|
|Default|	Active	|100.64.0.0/10	 | None	             |-|	-|
|Default|	Active	|192.168.0.0/16	 | None	             |-|	-|
|User   |	Active	|10.0.2.0/24	   | Virtual appliance | 10.0.1.10|	route-to-hub2|
|User   |	Active	|10.0.4.0/24	   | Virtual appliance | 10.0.1.10|	route-to-spoke2|
|User   |	Active	|10.0.3.0/24	   | Virtual appliance | 10.0.1.10|	route-to-spoke1|
|Default|	Invalid	|10.0.2.0/24	   | VNetGlobalPeering |-|	-|
|Default|	Invalid	|10.0.3.0/24	   | VNetGlobalPeering |-|	-|


#### vm-hub2-subnet2
|Source |	State	  |Address Prefixes| Next Hop Type	   |Next Hop Type IP Address |User Defined Route Name |
| ----- |:-------:|:---------------|:-----------------:|:-----------------------:|:----------------------:|
|Default|	Active  |	10.0.2.0/24    |Virtual network	   |-|	-|
|Default|	Active  |	0.0.0.0/0	     | Internet	         |-|	-|
|Default|	Active  |	10.0.0.0/8	   | None	             |-|	-|
|Default|	Active  |	100.64.0.0/10	 | None	             |-|	-|
|Default|	Active  |	192.168.0.0/16 | None	             |-|	-|
|User   |	Active  |	10.0.1.0/24	   |Virtual appliance  |	10.0.2.10	|route-to-hub1|
|User   |	Active  |	10.0.3.0/24	   |Virtual appliance  |	10.0.2.10	|route-to-spoke1|
|User   |	Active  |	10.0.4.0/24	   |Virtual appliance  |	10.0.2.10	|route-to-spoke2|
|Default|	Invalid |	10.0.1.0/24	   |VNetGlobalPeering  |-	|-|
|Default|	Invalid |	10.0.4.0/24	   |VNetGlobalPeering	 |-	|-|



## <a name="tcpdump"></a>3. Check the traffic transit through nva1 and nva2
Check the traffic between vm-spoke1 and vm-spoke2, with transit through nva1,nva2:

```console
[root@nva1 ~]# tcpdump -nq -ttt host 10.0.3.10 or host 10.0.4.10
[root@nva2 ~]# tcpdump -nq -ttt host 10.0.3.10 or host 10.0.4.10
```

Check the traffic between vm-hub1-subnet2 and vm-spoke1, with transit through nva1:

```console
[root@nva1 ~]# tcpdump -nq -ttt host 10.0.1.70 or host 10.0.3.10
```

Check the traffic between vm-hub1-subnet2 and vm-hub2-subnet2:

```console
[root@nva1 ~]# tcpdump -nq -ttt host 10.0.1.70 or host 10.0.2.70
[root@nva2 ~]# tcpdump -nq -ttt host 10.0.1.70 or host 10.0.2.70
```
Check the traffic between vm-hub1-subnet2 and vm-spoke2:

```console
[root@nva1 ~]# tcpdump -nq -ttt host 10.0.1.70 or host 10.0.4.10
[root@nva2 ~]# tcpdump -nq -ttt host 10.0.1.70 or host 10.0.4.10
```

To generate traffic between VMs you can use one of methods described in the [Traffic generation between two Azure VMs](https://github.com/fabferri/az-pattern/tree/master/00-traffic-between-2vms)


<!--Image References-->

[1]: ./media/network-diagram.png "network diagram"
[2]: ./media/flow.png "tcp flow transit from vm2 to vm3"

<!--Link References-->

