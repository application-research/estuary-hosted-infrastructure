# Welcome to Filecoin Data Infrastructure
Welcome!
## [Basics](admin/basics/index.md)
Basics covers some basics of operating and managing FDI, and common tasks and things you may want to do or monitor.

It covers the following topics: Connecting to FDI (Netbird), Proxmox (Groundfloor), staff desktops, DNS, DHCP, machine booting, monitoring, backups, security & patching.

## [Databases](admin/databases/index.md)
Databases contains a list of PostgreSQL databases and where they exist in the infrastructure, for both VM-hosted and Kubernetes hosted ("Postgres Operator") databases.

## [Kubernetes and CPI](admin/kubernetes/index.md)
The Kubernetes section covers: Logging into Kubernetes, getting Kubectl access, and managing Customer Provisioned Infrastructure (managing customers, creating new customers, editing existing customers)

## [Services](admin/services/index.md)
Services covers the various services hosted in FDI or that allow one to host things using FDI.

These include: Rancher, AWX, EstuaryV2 - Edge, Delta, Edge-URID, Carriers, Shuttle 12, API node

## [Storage](admin/storage/index.md)
Storage covers the four main storage systems in use at FDI, and how to care and feed for them.

These include: ZFS (ShuttleRescue), MooseFS, GarageHQ and Ceph.

## [Rescue](admin/rescue/index.md)
Rescue covers accessing the infrastructure when All Hope Is Lost, when you need to access things with no other easy way in, or when a physical Groundfloor node is having issues. It covers accessing machines using IPMI (out of band access for when the OS is not responding) and dialing in using Wireguard (for when the Netbird Bastion hosts are unavailable)

## [TLS](admin/tls/index.md)
TLS covers all things certificates, and troubleshooting steps for rotating certificates and the like.

* Managing TLS - Rotating TLS throughout the infrastructure with Wildcard-TLS-Playbook
    * Keeping certificates up to date
    * Adding new hosts to distribute TLS certificates to
* Managing TLS - Self-signed TLS for PostgreSQL clusters (etcd)

## [General and Specialised Guides](admin/guides/index.md)
These are guides that don't (yet) fit in elsewhere in the documentation.

* Diagnosing Pacemaker clusters including Prod HAProxy
* Ceph: Rebalancing Ceph OSDs
* Kubernetes: Fixing stuck Rook.io volumes which are preventing containers from starting
