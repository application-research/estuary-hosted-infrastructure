# Basics

## Proxmox
Proxmox is accessible at [https://proxmox.estuary.tech:8006]() using [these credentials](https://start.1password.com/open/i?a=4XNRW7JPXZEI7C7CEIAF27VTSQ&h=my.1password.com&i=i2yft7dcslm5trfq2t7taognuy&v=ki4skn3vacuvcqz3bcw532weou)

It provides a single-pane-of-glass view of everything that powers FDI, and should be considered the "first port of call" for most troubleshooting.

Within Proxmox, you can stop, start and reboot VMs, as well as migrate them with zero functional downtime between physical hosts, and access the "physical" console of each virtual machine (in other words, attach a monitor, keyboard and mouse to a virtual machine and diagnose what's happening with it).

Behind the scenes, [https://proxmox.estuary.tech:8006]() is powered by the physical node Vega. If Vega is down, you can access the other nodes - try [https://10.24.0.202:8006](), [https://10.24.0.203:8006]() etc.

## DNS
Our DNS is powered by Technitium DNS, and the machines prod-dns01, prod-dns02, prod-dns03.

These DNS servers are at 10.24.0.51, 10.24.0.52, 10.24.0.53 respectively.

Netbird (which you'll usually use to stay connected to FDI) automatically sets your machine to use these DNS servers so you can access internal resources.

## DHCP
Most core machines do not use DHCP for their addresses, but some do. DHCP is dependent on MAAS - Metal as a Service - which is an Estuary Bootstrap service. It runs primarily on prod-ebi-maas01.

## Booting machines (normally)
To start a machine, use Proxmox to start it. If it's stuck, check the console and try and look for clues as to why it is not booting. If no machines are booting (or if many machines are not booting) check Ceph's health in the Proxmox interface and make sure it's healthy.

## Creating, deleting and managing machines and machine definitions
From time to time, it's necessary to create new virtual machines. You can do this easily using the ProxMAAS repo and Vega.

* Check out the [ehi-proxmaas](https://github.com/application-research/ehi-proxmaas) repo somewhere you can work on it.
* Within the ehi-proxmaas project, look under `vars/production-ehi` for the **machine definitions** used to spawn machines.
* Create a machine definition
* Run the spawn.yml ansible playbook against that machine definition

For further details and the actual commands necessary, consult [the ProxMAAS documentation](proxmaas.md). 

To spawn machines using ProxMAAS using Vega (one of the Proxmox nodes), ssh to root@vega.estuary.tech, and `cd ehi-proxmaas`. 

To set up a local machine to be able to spawn machines, make sure Netbird is installed, then copy `vars/secrets.yml` from `root@vega.estuary.tech:/root/ehi-proxmaas/vars/secrets.yml` to your local copy of the repository, making sure to avoid committing the secrets.

## Monitoring and Logging
There are a number of things that need monitoring throughout the infrastructure. We have various tools for this purpose. Here's a few of them:

* Our [centralised monitoring solution](https://monitoring.estuary.tech) is provided by CheckMK Enterprise. A VM hosted on Vultr Cloud lives outside the infrastructure and monitors all of our hosts for issues, everything from dropped packets to disks filling and services not working. You can find the login for CheckMK [in the password manager](https://start.1password.com/open/i?a=4XNRW7JPXZEI7C7CEIAF27VTSQ&h=my.1password.com&i=o4oxrarr4tttbsz5ql2a3wxnpq&v=ki4skn3vacuvcqz3bcw532weou).

* The [MooseFS CGI monitor](http://mfsmaster.estuary.tech:9425) allows us to keep an eye on MooseFS and should be checked frequently for warnings and error messages and dead disks and such. Efforts are underway to integrate information from the MooseFS API into CheckMK and open source those checks.

* The [Ceph dashboard](https://proxmox.estuary.tech:8006/#v1:0:=node%2Faltair:4:38::::::38) allows us to keep an eye on Ceph and its health, which should be tended to carefully as Ceph underpins ***all VMs and all workloads running on FDI***

* [ArgoCD](https://argocd.estuary.tech) provides a view into Customer Provisioned Infrastructure (Phosphophyllite), and can be used to see at a glance whether any customers' Kubernetes resources appear "out of sync" with what they should be, which could indicate problems.

## Backups
There are two core backup services that run in FDI at the moment.

These are:

* pgBackRest @ prod-db-backup01 (for PostgreSQLs running as VMs)
* WAL-G @ Kubernetes [via GarageHQ] (for PostgreSQLs running as Kubernetes pods)

### WAL-G @ Kubernetes [via GarageHQ]
We run WAL-G to backup all PostgreSQL databases running in Kubernetes. It runs inside the `postgres` containers inside each PostgreSQL pod.

It backs up to GarageHQ using the `postgresql-backups` key pair and the https://s3.estuary.tech endpoint.

In order to check the status of backups for a particular cluster, find a pod within that cluster in Rancher, select the (⋮) and click `> Execute Shell` to get a shell inside the container, then run the following command inside it:
```
envdir "/run/etc/wal-e.d/env" wal-g backup-list
```

You should see something like this, with recent backups listed.
```
root@estuary-api-0:/home/postgres# envdir "/run/etc/wal-e.d/env" wal-g backup-list
name                          modified             wal_segment_backup_start
base_0000001A00001E510000000E 2023-07-06T11:29:59Z 0000001A00001E510000000E
base_0000001A00001E510000005C 2023-07-07T10:33:08Z 0000001A00001E510000005C
base_0000001B00001E51000000A0 2023-07-08T09:31:31Z 0000001B00001E51000000A0
base_0000001B00001E51000000D5 2023-07-09T10:26:17Z 0000001B00001E51000000D5
base_0000001B00001E520000000C 2023-07-10T11:20:18Z 0000001B00001E520000000C
base_0000001B00001E5200000044 2023-07-11T09:16:29Z 0000001B00001E5200000044
```

### pgBackRest
pgBackRest is currently set up to backup the EHI PostgreSQL cluster. It lives on `prod-db-backup01.estuary.tech`, and must remain functional. If the pgBackRest repo server (prod-db-backup01) is down, PostgreSQL has nowhere to archive WAL logs to and will collect them indefinitely, causing an eventual out-of-disk-space condition which will take down the EHI PostgreSQL cluster.

It currently backs up to Ceph, and depends on an external Ceph backup for backups to remain 100% safe.

!!! danger "That bears repeating!"
    pgBackRest being down or non-functional means the EHI database - which EstuaryV2 depends on heavily - will eventually run out of space and stop.

#### Checking on pgBackRest
There are two relevant commands for pgBackRest - `info` and `check`.

Check ensures that WAL archiving is working and sane, which is critical to backups working and continuing to work. Here's how you run it:
```
ubuntu@prod-db-backup01:~$ sudo -u pgbackrest pgbackrest --stanza=fdi_main --log-level-console=info check
2023-07-12 17:04:38.783 P00   INFO: check command begin 2.46: --exec-id=128661-f4c4ef14 --log-level-console=info --pg1-host=prod-ehi-db01.estuary.tech --pg2-host=prod-ehi-db02.estuary.tech --pg3-host=prod-ehi-db03.estuary.tech --pg1-path=/var/lib/postgresql/14/prod_ehi_db/ --pg2-path=/var/lib/postgresql/14/prod_ehi_db/ --pg3-path=/var/lib/postgresql/14/prod_ehi_db/ --pg1-port=5432 --pg2-port=5432 --pg3-port=5432 --repo1-cipher-pass=<redacted> --repo1-cipher-type=aes-256-cbc --repo1-path=/var/lib/pgbackrest --stanza=fdi_main
2023-07-12 17:04:42.393 P00   INFO: check repo1 (standby)
2023-07-12 17:04:42.394 P00   INFO: switch wal not performed because this is a standby
2023-07-12 17:04:42.394 P00   INFO: check repo1 configuration (primary)
2023-07-12 17:04:42.596 P00   INFO: check repo1 archive for WAL (primary)
2023-07-12 17:04:43.008 P00   INFO: WAL segment 000001110000052D000000CB successfully archived to '/var/lib/pgbackrest/archive/fdi_main/14-1/000001110000052D/000001110000052D000000CB-4285acb0ced7c37208bad415bbb53f82450b1aa0.gz' on repo1
2023-07-12 17:04:43.309 P00   INFO: check command end: completed successfully (4528ms)
```

Info allows you to check the status of backups and ensure there are recent complete backups of PostgreSQL available in case of a disaster. Here's how you run it:
```
ubuntu@prod-db-backup01:~$ sudo -u pgbackrest pgbackrest --stanza=fdi_main --log-level-console=info info
stanza: fdi_main
    status: ok
    cipher: aes-256-cbc

    db (current)
        wal archive min/max (14): 0000010500000352000000DD/000001110000052D000000CC

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

## Security & Patching