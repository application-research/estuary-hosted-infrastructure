
# Re-init a Postgresql Node


## Sometimes a postgresql node will fall out of sync like so:

```
ubuntu@prod-ehi-db01:~$ sudo /usr/local/bin/patronictl -c /etc/patroni/prod-ehi-db01.estuary.tech.yml list
+ Cluster: prod_ehi_db ------+------------+---------+--------------+-----+-----------+
| Member                     | Host       | Role    | State        |  TL | Lag in MB |
+----------------------------+------------+---------+--------------+-----+-----------+
| prod-ehi-db01.estuary.tech | 10.24.3.20 | Replica | running      | 266 |    349570 |
| prod-ehi-db02.estuary.tech | 10.24.3.21 | Replica | start failed |     |   unknown |
| prod-ehi-db03.estuary.tech | 10.24.3.22 | Leader  | running      | 269 |           |
+----------------------------+------------+---------+--------------+-----+-----------+
```


## The failing node must be re-initialized:

```
ubuntu@prod-ehi-db01:~$ sudo /usr/local/bin/patronictl -c /etc/patroni/prod-ehi-db01.estuary.tech.yml reinit prod_ehi_db
+ Cluster: prod_ehi_db ------+------------+---------+--------------+-----+-----------+
| Member                     | Host       | Role    | State        |  TL | Lag in MB |
+----------------------------+------------+---------+--------------+-----+-----------+
| prod-ehi-db01.estuary.tech | 10.24.3.20 | Replica | running      | 266 |    349575 |
| prod-ehi-db02.estuary.tech | 10.24.3.21 | Replica | start failed |     |   unknown |
| prod-ehi-db03.estuary.tech | 10.24.3.22 | Leader  | running      | 269 |           |
+----------------------------+------------+---------+--------------+-----+-----------+
Which member do you want to reinitialize [prod-ehi-db02.estuary.tech, prod-ehi-db01.estuary.tech]? []: prod-ehi-db02.estuary.tech
Are you sure you want to reinitialize members prod-ehi-db02.estuary.tech? [y/N]: y
Success: reinitialize for member prod-ehi-db02.estuary.tech
```


## This changes the state to 'creating replica':

```
pcadmin@workstation:~$ ssh prod-ehi-db01.estuary.tech -t "sudo patronictl -c /etc/patroni/prod-ehi-db01.estuary.tech.yml list"
+ Cluster: prod_ehi_db ------+------------+---------+------------------+-----+-----------+
| Member                     | Host       | Role    | State            |  TL | Lag in MB |
+----------------------------+------------+---------+------------------+-----+-----------+
| prod-ehi-db01.estuary.tech | 10.24.3.20 | Replica | creating replica |     |   unknown |
| prod-ehi-db02.estuary.tech | 10.24.3.21 | Replica | running          | 269 |         0 |
| prod-ehi-db03.estuary.tech | 10.24.3.22 | Leader  | running          | 269 |           |
+----------------------------+------------+---------+------------------+-----+-----------+
Connection to prod-ehi-db01.estuary.tech closed.
```

