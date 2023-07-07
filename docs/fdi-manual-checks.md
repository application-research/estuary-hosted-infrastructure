
# FDI Manual Checks

A long list of manual checks that can be done to examine all the important components in FDI.


## Bastion Servers

pcadmin@workstation:~$ for i in 01 02 03; do ssh "wings@prod-bastion${i}.estuary.tech" -t "sudo systemctl status netbird.service && exit"; done

pcadmin@workstation:~$ for i in 01 02 03; do ssh "wings@prod-bastion${i}.estuary.tech" -t "sudo journalctl -n 25 -u netbird.service && exit"; done


## Ceph

https://10.24.0.204:8006/#v1:0:=node%2Faltair:4:38::::::38

Check that Ceph is:
- OK
- Has at least 3 Monitors, Managers and Meta Data Servers
- Is not over 80% full
- Logs


## MooseFS

http://mfsmaster.estuary.tech:9425/mfs.cgi

Check that the MooseFS:

- masters are up!
- no missing chunks
- no errors on any Disk
- chunk servers are OK


## Estuary Status Page

https://status.estuary.tech/


## CheckMK

Examine:

https://monitoring.estuary.tech/estuary/check_mk/login.py

Green good, red bad! :)


## Database Clusters

$ ssh prod-ehi-db01.estuary.tech -t "sudo patronictl -c /etc/patroni/prod-ehi-db01.estuary.tech.yml list"
+ Cluster: prod_ehi_db ------+------------+---------+---------+----+-----------+-----------------+
| Member                     | Host       | Role    | State   | TL | Lag in MB | Pending restart |
+----------------------------+------------+---------+---------+----+-----------+-----------------+
| prod-ehi-db01.estuary.tech | 10.24.3.20 | Leader  | running | 76 |           | *               |
| prod-ehi-db02.estuary.tech | 10.24.3.21 | Replica | running | 76 |         0 | *               |
| prod-ehi-db03.estuary.tech | 10.24.3.22 | Replica | running | 76 |         0 | *               |
+----------------------------+------------+---------+---------+----+-----------+-----------------+

$ ssh prod-ebi-db01.estuary.tech -t "sudo patronictl -c /etc/patroni/prod-ebi-db01.estuary.tech.yml list"
+ Cluster: prod_ebi_db ------+-----------+---------+---------+----+-----------+
| Member                     | Host      | Role    | State   | TL | Lag in MB |
+----------------------------+-----------+---------+---------+----+-----------+
| prod-ebi-db01.estuary.tech | 10.24.3.1 | Leader  | running | 13 |           |
| prod-ebi-db02.estuary.tech | 10.24.3.2 | Replica | running |    |        16 |
| prod-ebi-db03.estuary.tech | 10.24.3.3 | Replica | running |    |        16 |
+----------------------------+-----------+---------+---------+----+-----------+

Examine the RAM usage:

$ for i in 01 02 03; do ssh "prod-ehi-db${i}.estuary.tech" -t "free -h"; done
$ for i in 01 02 03; do ssh "prod-ebi-db${i}.estuary.tech" -t "free -h"; done

Examine the disk space usage:

$ for i in 01 02 03; do ssh "prod-ehi-db${i}.estuary.tech" -t "df -h"; done
$ for i in 01 02 03; do ssh "prod-ebi-db${i}.estuary.tech" -t "df -h"; done


## HAProxy

$ for i in 01 02 03; do ssh "prod-haproxy${i}.estuary.tech" -t "sudo crm_mon -r -1 && exit"; done

$ for i in 01 02 03; do ssh "prod-haproxy${i}.estuary.tech" -t "sudo systemctl status corosync.service && exit"; done
  

## Delta

$ for i in 01 02 03 04 05 06 07 08; do ssh "prod-ehi-delta${i}.estuary.tech" -t "sudo systemctl --no-pager status delta.service && exit"; done


## Edge

$ for i in 01 02 03 04 05 06 07 08; do ssh "prod-ehi-edge${i}.estuary.tech" -t "sudo systemctl --no-pager status edge.service && exit"; done


## Edge-urdi

$ for i in 01 02 03 04 05 06 07 08; do ssh "prod-ehi-edge-urid${i}.estuary.tech" -t "sudo systemctl --no-pager status delta.service && exit"; done

