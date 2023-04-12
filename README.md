# estuary-hosted-infrastructure
The main repository for EHI. This will eventually link out to everything in the Estuary Hosted Infrastructure and walk you through deploying it start to finish in as automated a way as possible. It also serves as a collection point for Issues that don't necessarily fit with a specific component.

EHI is separated into stages, which you can mix and match at your desire but should usually be followed in order.

## Stage 0: Primitives
These are required to be set up before you can deploy EHI and "unfurl" the cluster.

## Stage 1: Groundfloor
Setting up Proxmox is currently a manual process, which can be done once the Primitives are in place.

## Stage 2: Estuary Bootstrap Infrastructure
Now that we have Proxmox installed, we can create and install some basic infrastructure. 

This will require manually installing some virtual machines, after which ProxMAAS can take over and deploy the rest of the infrastructure for you.

## Stage 3: Automatic Machine Provisioning
This stage establishes a full installation of ProxMAAS and allows you to start defining the rest of your infrastructure as a set of inventory folders.

## Stage 4: Main EHI Deployments + Quality of Life
Once the groundwork has been laid, on top of the groundfloor, you can deploy the Rest of Estuary™️
