
# Troubleshooting Guide

Note that the certificates this script generates MUST have the IP as a 'Subject Alternative Name', test the pubkey at this URL to verify it: https://www.sslshopper.com/certificate-decoder.html


Once all that is done, you should have working PostgreSQL via the loadbalancer:
```
root@prod-pg02:~# patronictl -c /etc/patroni/prod-pg02.estuary.tech.yml list
+ Cluster: main -----------+-------------+---------+---------+----+-----------+
| Member                   | Host        | Role    | State   | TL | Lag in MB |
+--------------------------+-------------+---------+---------+----+-----------+
| prod-pg01.estuary.tech | 10.1.11.119 | Replica | running |  1 |         0 |
| prod-pg02.estuary.tech | 10.1.11.118 | Leader  | running |  1 |           |
| prod-pg03.estuary.tech | 10.1.11.117 | Replica | running |  1 |         0 |
+--------------------------+-------------+---------+---------+----+-----------+
```

Check if you can log into the postgresql database.
```
root@estuary-pg01:~# psql -h 10.1.3.1 -U postgres -W
Password:
psql (14.5 (Debian 14.5-2.pgdg110+2))
Type "help" for help.

postgres=#
```

## Stats Page

If you're using this playbook [with HAProxy](https://github.com/application-research/haproxy-cluster-playbook) you can visit the loadbalancer's IP on port 8080 to see a stats page:

http://prod-haproxy01.estuary.tech:8080/

This stats page shows you which node is the current leader (shown in green).

## TLS Secrets

Keep a backup of the certificate material from the secure box, and (if needed) shut down and destroy the secure box once the material has been safely backed up and verified.
