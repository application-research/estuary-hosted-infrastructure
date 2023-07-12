# Intro
Filecoin Data Infrastructure is the physical and digital infrastructure that powers Filecoin Data Tools.

It is a number of things:

* an open source project and blueprint for SPs and other parties that allows you to build your own private cloud ("an FDI")
* a hosting solution for all FDT customers and Estuary infrastructure

## Open source
Yes, we're open source. You can find the code in the following places, mainly:

### Core repos
Here are the core repositories that make up FDI

* [https://github.com/application-research]() - Main GitHub organisation
* [https://github.com/application-research/estuary-hosted-infrastructure]() - Main "EHI" repo, contains this documentation and all open source infrastructure code
* [https://github.com/application-research/estuary-hosted-infrastructure-private]() - Encrypted Git copy of relevant secrets and variables that cannot be shared publicly
* [https://github.com/application-research/ehi-proxmaas]() - "ProxMAAS" repo, where all machine deployments are currently defined and a tool that we use to deploy machines automatically

### Ansible repos
We love Ansible, and all our Ansible is open source. It's a little rough around the edges and could use some documentation love - PRs welcome!

* [https://github.com/application-research/edge-playbook]()
* [https://github.com/application-research/delta-playbook]()
* [https://github.com/application-research/delta-dm-playbook]()
* [https://github.com/application-research/edge-urid-playbook]()
* [https://github.com/application-research/wildcard-tls-playbook]()
* [https://github.com/application-research/haproxy-cluster-playbook]()
* [https://github.com/application-research/postgresql-cluster-playbook]()

