# estuary-hosted-infrastructure
The main repository for EHI. This will eventually link out to everything in the Estuary Hosted Infrastructure and walk you through deploying it start to finish in as automated a way as possible. It also serves as a collection point for Issues that don't necessarily fit with a specific component.

EHI is separated into stages, which you can mix and match at your desire but should usually be followed in order.

To run everything in one big sweep, copy `vars/secrets.yml.dist` to `vars/secrets.yml`, fill it out, then run ansible-playbook main.yml in the root of this repository.

## Stage 0: Primitives
These are required to be set up before you can deploy EHI and "unfurl" the cluster.

* s00-01-routerconfigs/ - Create router configuration for the MikroTik core router. (TODO: Include DHCP config for *only* the BMC network)
* s00-02-switchconfigs/ - Create core switch and BMC switch configuration rules. (TODO: Configure all MAAS IPs as DHCP helper IPs on core-sw)
* s00-03-physical/ - Physically cable everything together and bring up the network
* s00-04-disable-lacp/ - Temporarily disable LACP for the Groundfloor Proxmox nodes so they can boot with one network link open.

## Stage 1: Groundfloor
Setting up Proxmox is currently a manual process, which can be done once the Primitives are in place.

* s01-01-proxmox/ - Install, configure and cluster Proxmox VE, bringing up the basic Groundfloor infrastructure.
* s01-02-ceph/ - Bring up the Ceph cluster, adopting all disks.
* s01-03-moosefs/ - Bring up MooseFS (Pro), creating a highly available data lake.

## Stage 2: Estuary Bootstrap Infrastructure
Now that we have Proxmox installed, we can create and install some basic infrastructure. 

* s02-01-genesis/ - Creates an LXC container which will run MAAS and is used to spawn and build out the rest of the cluster.
  * The genesis machine will take the first MAAS IP address, then once the second and third MAAS machines are created, it will remove MAAS from itself and move itself to a different IP address (10.24.137.137) to finish the rest of its work.
  * The genesis machine will use ProxMAAS to instrument Proxmox and build out all the required VMs for the EBI deployment.
* s02-02-dns/ - Creates a Technitium DNS cluster and upserts all of our DNS configuration into it
* s02-03-tls-bastion/ - Creates the TLS/HTTPS certificates needed to authenticate various things, a prerequisite for our HAProxy setup.
* s02-04-haproxy/ - Creates the initial HAProxy installation and cluster.
* s02-05-ebi-db/ - Creates the EBI PostgreSQL cluster for Nautobot & AWX to run on.
* s02-06-ebi/ - Runs the ehi-bootstrap-playbook playbook using Genesis, to provide Rancher and AWX services to the cluster.
* s02-07-bastion/ - Creates the Bastion infrastructure through Netbird to allow Estuary staff to access the EHI network.
* s02-08-nautobot/ - Creates a highly available Nautobot installation and sets up ChatOps via Slack.

## Stage 3: Automatic Machine Provisioning
This stage establishes a full installation of ProxMAAS and allows you to start defining the rest of your infrastructure as a set of inventory folders.

* s03-01-maas/ - Creates the MAAS infrastructure, migrates a copy of the MAAS databases to itself, creates the second and third MAAS nodes using that new database, migrates Genesis to 10.24.137.137, then removes MAAS from it and creates the proper first MAAS node at its correct IP and sets up the final MAAS nodes.

## Stage 4: Main EHI Deployments + Quality of Life
Once the groundwork has been laid, on top of the groundfloor, you can deploy the Rest of Estuary™️

* s04-01-ehi-db/ - Deploys the main EHI PostgreSQL cluster for various databases (other than API) to run on.
* s04-02-ehi-api-db/ - Deploys the EHI PostgreSQL cluster for the Estuary API to run on.
* s04-03-ehi-api-nsq/ - Deploys the EHI NSQ cluster for the Estuary API to run on.
* s04-04-ehi-api/ - Deploys the Estuary API cluster.
* s04-05-ehi-edge/ - Deploys the Estuary Edge cluster.
* s04-06-phos-k8s/ - Deploys the Phosphophyllite Kubernetes cluster.

# TODOs/coming soon
* A script that allows you to edit the VLANs and IP addressing schemes of the entire EHI stack and output a new set of instructions.
