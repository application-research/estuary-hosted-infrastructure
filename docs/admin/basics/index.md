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
* Follow the 

### To work from Vega
To spawn machines using ProxMAAS using Vega (one of the Proxmox nodes), ssh root@vega.estuary.tech 

### To work from your local machine

## Monitoring
## Backups
There are two core backup services that run in FDI at the moment.

These are:

* pgBackRest @ prod-db-backup01 (for PostgreSQLs running as VMs)
* WAL-G @ Kubernetes [via GarageHQ] (for PostgreSQLs running as Kubernetes pods)

### pgBackRest



## Security & Patching