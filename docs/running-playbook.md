
# A guide for running this playbook outside of AWX

## Setup Aliases

Configure the following bash aliases:

```
$ cat ~/.bashrc
...
# Aliases
alias ansible-ehi='ansible -u ubuntu -i ~/projects/estuary-hosted-infrastructure/inventories/production-ehi/ -i ~/projects/estuary-hosted-infrastructure-private/inventories/production-ehi/'
alias ansible-playbook-ehi='ansible-playbook -u ubuntu -i ~/projects/estuary-hosted-infrastructure/inventories/production-ehi/ -i ~/projects/estuary-hosted-infrastructure-private/inventories/production-ehi/'
```


## Git clone the two necessary repos with submodules

```
$ cd ~/projects/
$ git clone git@github.com:application-research/estuary-hosted-infrastructure.git --recurse-submodules
$ git clone git@github.com:application-research/estuary-hosted-infrastructure-private.git --recurse-submodules
```


## Create any necessary Vault secrets locally on your machine 

We recommend deleting them once done each time, and only doing this on a machine with full disk encryption.

```
$ mkdir -p ~/.ehi-vault
$ nano ~/.ehi-vault/delta
$ nano ~/.ehi-vault/loadbalancer
$ nano ~/.ehi-vault/moosefs
$ nano ~/.ehi-vault/postgresql
$ nano ~/.ehi-vault/proxmaas
$ nano ~/.ehi-vault/tls-materials
$ nano ~/.ehi-vault/tls-bastion
```


## Run the main playbook with every variable file!

```
$ ansible-playbook-ehi main.yml \
--vault-id delta@~/.ehi-vault/delta \
--vault-id loadbalancer@~/.ehi-vault/loadbalancer \
--vault-id moosefs@~/.ehi-vault/moosefs \
--vault-id postgresql@~/.ehi-vault/postgresql \
--vault-id proxmaas@~/.ehi-vault/proxmaas \
--vault-id tls-bastion@~/.ehi-vault/tls-bastion \
--vault-id tls-materials@~/.ehi-vault/tls-materials
```


## Run a specific section/inventory

For example, running ehi-proxmaas's "spawn.yml" against a specific inventory (dev-haproxy0[1-3]):

```
$ ansible-playbook-ehi playbooks/ehi-proxmaas/spawn.yml --vault-id proxmaas@~/.ehi-vault/proxmaas  --extra-vars "machine_details=dev-haproxy"
```

