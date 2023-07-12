
# Examine FDI Backups


## Check GarageHQ Backups for k8s DBs

- Login to Rancher

- Select 'Production Estuary' namespace in top right corner

- Go 'Workloads' > 'Pods'

- Open shell in either estuary-api-0 or estuary-api-1 pods:

Enter this command to examine backups:

`envdir "/run/etc/wal-e.d/env" wal-g backup-list`


## Examine pgBackRest Backups for VM DBs


### List pgbackrest config + stanza names

`ubuntu@prod-db-backup01:~$ sudo cat /etc/pgbackrest/pgbackrest.conf`


### Check Backups are functional for stanza fdi_main

`ubuntu@prod-db-backup01:~$ sudo -u pgbackrest pgbackrest --stanza=fdi_main --log-level-console=info check`


### Check backup history for stanza fdi_main

```
ubuntu@prod-db-backup01:~$ sudo -u pgbackrest pgbackrest --stanza=fdi_main --log-level-console=info info
stanza: fdi_main
    status: ok
    cipher: aes-256-cbc

    db (current)
        wal archive min/max (14): 0000010500000352000000DD/0000010900000362000000A4

        full backup: 20230618-031405F
            timestamp start/stop: 2023-06-18 03:14:05 / 2023-06-18 03:30:27
            wal start/stop: 0000010500000352000000DD / 0000010500000352000000DD
            database size: 211.9GB, database backup size: 211.9GB
            repo1: backup set size: 61.7GB, backup size: 61.7GB

        full backup: 20230625-031405F
            timestamp start/stop: 2023-06-25 03:14:05 / 2023-06-25 03:29:58
            wal start/stop: 000001080000035A000000C4 / 000001080000035A000000C4
            database size: 213.1GB, database backup size: 213.1GB
            repo1: backup set size: 62GB, backup size: 62GB
```
