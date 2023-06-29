
# How to restart HA nodes/services?

All the services in FDI are designed to be highly-available, but still care must be taken when restarting certain machines/services.

For most HA triplets, each node can just be updated/rebooted one by one.


## Edge and Delta

The status of these services should be observed on your HAProxy's status page during these restarts. For example: https://prod-haproxy01.estuary.tech:8443/

[IMAGE]

These services are easy to reset, simply loop over them:
```
$ for i in 01 02 03 04 05 06 07 08; do ssh "prod-ehi-delta${i}.estuary.tech" -t "sudo apt update && sudo apt upgrade -y && sudo reboot"; done
$ for i in 01 02 03 04 05 06 07 08; do ssh "prod-ehi-edge${i}.estuary.tech" -t "sudo apt update && sudo apt upgrade -y && sudo reboot"; done
```


## Postgresql/Patroni clusters:

1) First use patronictl to observe the current leader:
```
$ ssh prod-ehi-db01.estuary.tech -t "sudo patronictl -c /etc/patroni/prod-ehi-db01.estuary.tech.yml list"
+ Cluster: prod_ehi_db ------+------------+---------+---------+-----+-----------+
| Member                     | Host       | Role    | State   |  TL | Lag in MB |
+----------------------------+------------+---------+---------+-----+-----------+
| prod-ehi-db01.estuary.tech | 10.24.3.20 | Replica | running | 265 |         0 |
| prod-ehi-db02.estuary.tech | 10.24.3.21 | Leader  | running | 265 |           |
| prod-ehi-db03.estuary.tech | 10.24.3.22 | Replica | running | 265 |         0 |
+----------------------------+------------+---------+---------+-----+-----------+
```

2) Patch one node at a time except for the leader, restarting as you go:
```
$ ssh prod-ehi-db01.estuary.tech -t "sudo apt update && sudo apt upgrade -y && sudo reboot"
$ ssh prod-ehi-db03.estuary.tech -t "sudo apt update && sudo apt upgrade -y && sudo reboot"
```

3) Examine if the other (Replica) nodes have come back online:
```
$ ssh "prod-ehi-db01.estuary.tech" -t "sudo patronictl -c /etc/patroni/prod-ehi-db01.estuary.tech.yml list"
+ Cluster: prod_ehi_db ------+------------+---------+---------+-----+-----------+
| Member                     | Host       | Role    | State   |  TL | Lag in MB |
+----------------------------+------------+---------+---------+-----+-----------+
| prod-ehi-db01.estuary.tech | 10.24.3.20 | Replica | running | 265 |         0 |
| prod-ehi-db02.estuary.tech | 10.24.3.21 | Leader  | running | 265 |           |
| prod-ehi-db03.estuary.tech | 10.24.3.22 | Replica | running | 265 |         0 |
+----------------------------+------------+---------+---------+-----+-----------+
```

4) Then do a planned switchover from leader to follower and finally patch the remaining machine:
```
$ ssh prod-ehi-db01.estuary.tech -t "sudo patronictl -c /etc/patroni/prod-ehi-db01.estuary.tech.yml switchover --candidate prod-ehi-db01.estuary.tech"
Current cluster topology
+ Cluster: prod_ehi_db ------+------------+---------+---------+-----+-----------+
| Member                     | Host       | Role    | State   |  TL | Lag in MB |
+----------------------------+------------+---------+---------+-----+-----------+
| prod-ehi-db01.estuary.tech | 10.24.3.20 | Replica | running | 265 |         0 |
| prod-ehi-db02.estuary.tech | 10.24.3.21 | Leader  | running | 265 |           |
| prod-ehi-db03.estuary.tech | 10.24.3.22 | Replica | running | 265 |         0 |
+----------------------------+------------+---------+---------+-----+-----------+
Primary [prod-ehi-db02.estuary.tech]:            
When should the switchover take place (e.g. 2023-06-29T13:23 )  [now]: 
Are you sure you want to switchover cluster prod_ehi_db, demoting current leader prod-ehi-db02.estuary.tech? [y/N]: y
2023-06-29 12:27:32.40370 Successfully switched over to "prod-ehi-db01.estuary.tech"
+ Cluster: prod_ehi_db ------+------------+---------+----------+-----+-----------+
| Member                     | Host       | Role    | State    |  TL | Lag in MB |
+----------------------------+------------+---------+----------+-----+-----------+
| prod-ehi-db01.estuary.tech | 10.24.3.20 | Leader  | running  | 265 |           |
| prod-ehi-db02.estuary.tech | 10.24.3.21 | Replica | stopping |     |   unknown |
| prod-ehi-db03.estuary.tech | 10.24.3.22 | Replica | running  | 265 |         0 |
+----------------------------+------------+---------+----------+-----+-----------+
$ ssh prod-ehi-db02.estuary.tech -t "sudo apt update && sudo apt upgrade -y && sudo reboot"
```

5) Run patronictl list again to examine if all the nodes are still working.


## Other HA Services

The status of these services should be observed on your HAProxy's status page during these restarts. For example: https://prod-haproxy01.estuary.tech:8443/

[IMAGE]

```
$ for i in 01 02 03; do ssh "prod-fwscdn${i}.estuary.tech" -t "sudo apt update && sudo apt upgrade -y && sudo reboot"; done
$ for i in 01 02 03; do ssh "prod-carrier${i}.estuary.tech" -t "sudo apt update && sudo apt upgrade -y && sudo reboot"; done
```
