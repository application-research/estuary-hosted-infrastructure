
# ProxMAAS

ProxMAAS allows for you to define virtual hosts in variable files and then create them automatically. New hosts will be created inside Proxmox, booted, then automatically installed and managed by MAAS.

## Creating machine definitions (for spawning VMs)

1) In the ehi-proxmaas playbook, create a new machine definition (variable) file that defines the new hosts you want to make:
```
user@workstation:~/projects/ehi-proxmaas$ cat vars/production-ehi/prod-garage.yml 
---
# Machine specs
scsi_disk_layout:
  scsi0: "vm-storage:50,format=raw"
amount_of_memory: "16384"
number_of_vcpus: "64"
ip_range:
  - start: 10.24.100.1
    end: 10.24.100.2
starting_number: "1"
vm_name: staff-schreck
vm_description: "Schreck's personal machines"
```

Here's a rough breakdown of what each of those do:

* scsi_disk_layout: What disks the VM should have. scsi0 is always the root disk, vm-storage represents the Proxmox storage pool that will be used (always use vm-storage), 300 represents the number of gibibytes (GiB) assigned to that disk, and the rest are common formatting options which you should leave alone unless told otherwise. You can specify additional disks as scsi1, scsi2 etc in the same format.

* amount_of_memory: The amount of memory for the VM, specified in mebibytes (MiB). Should always be a multiple of 1024.

* number_of_vcpus: The amount of virtual CPUs assigned to the VM. This cannot exceed 96 due to physical limitations, but should usually never exceed 48, and should always be a multiple of 2.

* ip_range: This controls both what IP addresses are assigned, as well as **how many machines will be spawned**. You can define a single machine per definition, or define an IP range containing multiple machines. They will be named according to the number of total IPs + the starting_number you specify.

* starting_number: The starting number for the set of VMs. Setting this to "1" will result in machines being spawned as "staff-schreck01, staff-schreck02 ...", setting it to a higher number will result in the VMs starting at that higher number.

* vm_name: The name to assign to the VM in DNS, MAAS and Proxmox.

* vm_description: A useful description that will appear in Proxmox explaining what the VM is and does.

Once you have created **and committed** your machine definition...

2) Run the spawn.yml playbook in ProxMAAS and define a {{ machine_details }} variable like so:
```
user@workstation:~/projects/ehi-proxmaas$ ansible-playbook spawn.yml --extra-vars "machine_details=prod-garage"
```

## Destroying VMs

The process for destroying VMs is quite similar, but is interactive. Prior to destroying VMs, make sure they are powered off.

```
user@workstation:~/projects/ehi-proxmaas$ ansible-playbook destroy.yml
...

TASK [Set the VM name for the VMs] **************************************************************************************************************************************
[Set the VM name for the VMs]
Enter the VM name for the VMs you want to destroy. PERMANENTLY!:
prod-garage
...
TASK [Set the starting number for the VMs] ******************************************************************************************************************************
[Set the starting number for the VMs]
Enter the starting number for the VMs (e.g. 01 if it's a new deployment, 04 if there are already 3 machines, etc.):
01
...
TASK [Set the starting number for the VMs] ******************************************************************************************************************************
[Set the starting number for the VMs]
Enter the number of VMs you want to DESTROY:
1
...
TASK [Ask the user if they wish to continue] ****************************************************************************************************************************
[Ask the user if they wish to continue]
Are you completely sure you wish to continue? (yesiamsure/n):
yesiamsure
```
