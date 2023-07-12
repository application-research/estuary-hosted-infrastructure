
# Quick Start Guide

1) Install the Ansible requirements:

`$ ansible-galaxy install -r requirements.yml`


2) This playbook requires a 'tls_bastion' server to be defined in the inventory, this is a server you're using exclusively for certificates/challenges etc. Using localhost is also an option if you don't have a tls_bastion server:
```
# add tls_bastion as localhost
[tls_bastion]
localhost ansible_connection=local
```


3) You must define a seperate inventory group for the {{ postgresql_cluster_name }} variable, for example if it's:
`postgresql_cluster_name: prod_estuary`

Means you'll need to configure a group for it in your inventory/hosts file like so:
```
[prod_estuary:children]
etcd_master
```


4) Define these extra variables in your ./inventories/pchq/group_vars/all.yml file:
```
internal_subnet: "10.1.1.0/16"                 # The internal subnet of your network
postgresql_cluster_name: prod_estuary        
patroni_postgresql_version: 15
patroni_install_haproxy: false
patroni_system_group: postgres
patroni_install_watchdog_loader: false         # This is needed as it disables a currently broken section
patroni_etcd_hosts: "{% for host in groups[postgresql_cluster_name] %}{{ hostvars[host]['inventory_hostname'] }}:2379{% if not loop.last %},{% endif %}{% endfor %}"  # yamllint disable-line rule:line-length
patroni_etcd_protocol: https
patroni_etcd_cacert: "/var/lib/patroni.pki/ca.pem"
patroni_etcd_cert: "/var/lib/patroni.pki/{{ inventory_hostname }}.pem"
patroni_etcd_key: "/var/lib/patroni.pki/{{ inventory_hostname }}-key.pem"
patroni_postgresql_connect_address: "{{ ansible_ens18.ipv4.address }}:5432"
patroni_restapi_connect_address: "{{ ansible_ens18.ipv4.address }}:{{ patroni_restapi_port }}"

patroni_postgresql_pg_hba:                                              # This section defines connection permissions between the postgresql hosts
  - { type: "local", database: "all", user: "all", method: "peer" }
  #- { type: "host", database: "all", user: "postgres", address: "{{ internal_subnet }}", method: "scram-sha-256" }
  - { type: "host", database: "replication", user: "{{ patroni_replication_username }}", address: "{{ internal_subnet }}", method: "scram-sha-256" } # Allow Patroni replication
  - { type: "host", database: "replication", user: "{{ patroni_replication_username }}", address: "127.0.0.1/32", method: "scram-sha-256" } # Allow local Patroni access
```


5) Run the playbook against your chosen inventory:
`$ ansible-playbook -v -i ./inventories/prod site.yml`

or with an escalation password if needed:
`$ ansible-playbook -i ./inventories/prod site.yml --ask-become-pass`
