
# How to restart HA nodes/services?

All the services in FDI are designed to be highly-available, but still care must be taken when restarting certain machines/services.

For most HA triplets, each node can just be updated/rebooted one by one.


## Edge and Delta

The status of these services should be observed on your HAProxy's status page during these restarts. For example: https://prod-haproxy01.estuary.tech:8443/

![Screenshot from 2023-06-29 20-55-41](https://github.com/application-research/estuary-hosted-infrastructure/assets/29645145/dd07fc4d-6b82-4ddf-b4bc-8abcf6c8a3e0)

These services are easy to reset, simply loop over them:
```
$ for i in 01 02 03 04 05 06 07 08; do ssh "prod-ehi-delta${i}.estuary.tech" -t "sudo apt update && sudo apt upgrade -y && sudo systemctl stop delta.service && sudo reboot"; done
$ for i in 01 02 03 04 05 06 07 08; do ssh "prod-ehi-edge${i}.estuary.tech" -t "sudo apt update && sudo apt upgrade -y && sudo systemctl stop edge.service && sudo reboot"; done
$ for i in 01 02 03 04 05 06 07 08; do ssh "prod-ehi-edge-urid${i}.estuary.tech" -t "sudo apt update && sudo apt upgrade -y && sudo systemctl stop edge.service && sudo reboot"; done
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


6) After completion also be sure to update the prod-backup-db01 server as it needs to have the exact same version of the pgBackRest package to function:

`$ ssh prod-db-backup01.estuary.tech -t "sudo apt update && sudo apt upgrade -y && sudo reboot"`


## Other HA Services

The status of these services should be observed on your HAProxy's status page during these restarts. For example: https://prod-haproxy01.estuary.tech:8443/

![Screenshot from 2023-06-29 21-04-29](https://github.com/application-research/estuary-hosted-infrastructure/assets/29645145/8d7bf795-060d-460d-b809-fb5b0398db83)

```
$ for i in 01 02 03; do ssh "prod-fwscdn${i}.estuary.tech" -t "sudo apt update && sudo apt upgrade -y && sudo reboot"; done
$ for i in 01 02 03; do ssh "prod-carrier${i}.estuary.tech" -t "sudo apt update && sudo apt upgrade -y && sudo reboot"; done
$ for i in 01 02 03 04 05; do ssh "prod-nsqlookupd${i}.estuary.tech" -t "sudo apt update && sudo apt upgrade -y && sudo reboot"; done
```
