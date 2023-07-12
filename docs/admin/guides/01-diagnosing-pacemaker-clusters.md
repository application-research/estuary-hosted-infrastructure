# Diagnosing Pacemaker clusters
We run a number of Pacemaker clusters, but the primary one to keep an eye on is the prod-haproxy cluster.

This document is written about that cluster, but learnings from this one can be applied to other Pacemaker clusters (and to a lesser extent Proxmox, which uses Corosync, the same library Pacemaker uses, to keep its cluster members in sync and in quorum).

## Pacemaker's role in FDI
Pacemaker moves the virtual IP addresses 10.24.0.[2,3,4] around as needed, across the three Production HAProxy hosts. This allows us to use haproxy.estuary.tech to refer to the virtual IP addresses, and when a host fails or needs to be taken down for maintenance, the remaining two HAProxy hosts take over any IPs that were served by that third node and continue service like nothing happened.

The Production HAProxy boxes run an open source load balancer (HAProxy) that allows proxying traffic of many kinds, everything from S3 gateways to PostgreSQL traffic and Kubernetes. Most of the lifeblood of Estuary and FDT runs through the HAProxies. The multiple virtual IP addresses also serve the purpose of load balancing traffic across the HAProxies, splitting it roughly evenly using round-robin DNS.

## Checking the status of the cluster
To check the status of the Pacemaker cluster, use `sudo pcs status` on a haproxy node.

```
ubuntu@prod-haproxy01:~$ sudo pcs status
Cluster name: loadbalancer
Cluster Summary:
  * Stack: corosync
  * Current DC: prod-haproxy02.estuary.tech (version 2.1.2-ada5c3b36e2) - partition with quorum
  * Last updated: Wed Jul 12 10:59:45 2023
  * Last change:  Mon Jul 10 23:33:06 2023 by hacluster via crmd on prod-haproxy02.estuary.tech
  * 3 nodes configured
  * 6 resource instances configured

Node List:
  * Online: [ prod-haproxy01.estuary.tech prod-haproxy02.estuary.tech prod-haproxy03.estuary.tech ]

Full List of Resources:
  * virtual_ip_1	(ocf:heartbeat:IPaddr2):	 Started prod-haproxy02.estuary.tech
  * virtual_ip_2	(ocf:heartbeat:IPaddr2):	 Started prod-haproxy03.estuary.tech
  * virtual_ip_3	(ocf:heartbeat:IPaddr2):	 Started prod-haproxy01.estuary.tech
  * Clone Set: loadbalancer-clone [loadbalancer]:
    * Started: [ prod-haproxy01.estuary.tech prod-haproxy02.estuary.tech prod-haproxy03.estuary.tech ]

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled
```

You'll want to see that all 3 virtual_ip resources are being served by one node each, and that loadbalancer-clone is running on any nodes that are alive. This command will also tell you which nodes are up or down or in maintenance and allow you to take action accordingly.

## Debugging when things go wrong
If there are resources which have failed to stop or start in the cluster:

* `sudo pcs resource cleanup`

If a node is out of sync with the rest of the cluster

* `sudo systemctl restart corosync && sudo systemctl restart pcsd`

Standard Pacemaker debugging applies - try to get at least two nodes to appear and be healthy so that resources can be served.