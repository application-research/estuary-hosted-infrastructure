
# A guide for running this playbook outside of AWX


## Setup aliases

This makes it a lot simpler to run playbooks against the 2 repositories/inventories we use:
```
$ cat ~/.bash_aliases
# Aliases
alias ansible-ehi='ansible -u ubuntu -i ~/projects/estuary-hosted-infrastructure/inventories/production-ehi/ -i ~/projects/estuary-hosted-infrastructure-private/inventories/production-ehi/'
alias ansible-playbook-ehi='ansible-playbook -u ubuntu -i ~/projects/estuary-hosted-infrastructure/inventories/production-ehi/ -i ~/projects/estuary-hosted-infrastructure-private/inventories/production-ehi/'
```


## Running the playbooks

1) Git clone the two necessary repos with submodules
```
$ cd ~/projects/
$ git clone git@github.com:application-research/estuary-hosted-infrastructure.git --recurse-submodules
$ git clone git@github.com:application-research/estuary-hosted-infrastructure-private.git --recurse-submodules
```

2) Create any necessary Vault secrets locally on your machine (WARNING: we recommend deleting these once done each time, and only doing this on a machine with full disk encryption)
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

3) Run the main playbook with every variable file!
```
$ cd ~/projects/estuary-hosted-infrastructure
$ ansible-playbook-ehi main.yml \
--vault-id delta@~/.ehi-vault/delta \
--vault-id loadbalancer@~/.ehi-vault/loadbalancer \
--vault-id moosefs@~/.ehi-vault/moosefs \
--vault-id postgresql@~/.ehi-vault/postgresql \
--vault-id proxmaas@~/.ehi-vault/proxmaas \
--vault-id tls-bastion@~/.ehi-vault/tls-bastion \
--vault-id tls-materials@~/.ehi-vault/tls-materials
```


## How can I only run a specific section/inventory? 

For example, running ehi-proxmaas's "spawn.yml" against a specific inventory (dev-haproxy0[1-3]):

```
$ cd ~/projects/estuary-hosted-infrastructure
$ ansible-playbook-ehi playbooks/ehi-proxmaas/spawn.yml --vault-id proxmaas@~/.ehi-vault/proxmaas  --extra-vars "machine_details=dev-haproxy"
```

Or run playbooks/logtail.yml against dev-demo[01:03], first make sure you have defined the target inventory in your hosts file:
```
$ cat ~/projects/estuary-hosted-infrastructure/inventories/production-ehi/hosts
[dev_demo]
dev-demo[01:03].estuary.tech

[logtail_clients:children]
dev_demo
```
Then run a playbook against it:
```
$ cd ~/projects/estuary-hosted-infrastructure
$ ansible-playbook-ehi playbooks/logtail-playbook/disconnect.yml
```


## Encrypt a variable with a vault password (and vault-id):
```
$ ansible-vault encrypt_string 'strong-api-token-string' --name 'logtail_api_key' --vault-id proxmaas@prompt
New vault password (proxmaas): 
Confirm new vault password (proxmaas): 
Encryption successful
logtail_api_key: !vault |
          $ANSIBLE_VAULT;1.2;AES256;proxmaas
          11055633918770429142902173396911421216287390132204845285229540464063184442026986
          66859934371969282116487131834498694708145286012139144133895520666382339655331443
          57499947727087977380676419368809974229798141346440802729390819747629378816477878
          52896524828670163974022167202826172166235988258440501501293477735999168369912972
          53596788072402179773851957224793713770540977348438536151435086987057
```