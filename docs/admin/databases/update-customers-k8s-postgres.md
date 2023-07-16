
## Update customers postgres/edge/delta

It's hard to see in Rancher but there's logs indicating DB failure...

https://rancher.estuary.tech/dashboard/c/c-m-xvjqhpz7/explorer/pod

Apparently we need to update that customers postgres and edge/delta versions...


1) https://git.estuary.tech/fws-customers/customer-est-ana/src/branch/main/kubernetes/Chart.yaml
edit this file, but locally, in your own git checkout of it(edited)


2) bump it to the latest postgresql and latest edge, look here if you need a reference for what the latest is: https://github.com/application-research/fws-cpi-helm
(cpi-edge-0.9.5.tgz tells you Edge's latest is 0.9.5, and latest postgresql is 0.6.0)


3) so, edit the repo, changing Chart.yaml to have the newer versions
bump "version" on the chart itself as appropriate - if any of the charts changed a Y value in vX.Y.Z (ie: did more than a patch), bump the Y value on the chart
so ana's chart should go from 0.1.1 to 0.2.0


4) then before you commit it, cd to the kubernetes folder in the repo and run helm dependency update:

```
~/customer-est-ana/kubernetes$ helm dependency update
Getting updates for unmanaged Helm repositories...
...Successfully got an update from the "https://application-research.github.io/fws-cpi-helm" chart repository
...Successfully got an update from the "https://application-research.github.io/fws-cpi-helm" chart repository
Saving 2 charts
Downloading cpi-edge from repo https://application-research.github.io/fws-cpi-helm
Downloading cpi-postgresql from repo https://application-research.github.io/fws-cpi-helm
Deleting outdated charts
```

5) then commit all changes, and wait for ArgoCD to roll it out

pcadmin@workstation:~/MyVault/Estuary_LIVE/customer-est-ana$ git add .
pcadmin@workstation:~/MyVault/Estuary_LIVE/customer-est-ana$ git commit -am'update versions of edge/postgres'
[main 5a1d86a] update versions of edge/postgres
 7 files changed, 10 insertions(+), 7 deletions(-)
 create mode 100644 .vscode/settings.json
 delete mode 100644 kubernetes/charts/cpi-edge-0.8.1.tgz
 create mode 100644 kubernetes/charts/cpi-edge-0.9.5.tgz
 delete mode 100644 kubernetes/charts/cpi-postgresql-0.2.2.tgz
 create mode 100644 kubernetes/charts/cpi-postgresql-0.6.0.tgz
pcadmin@workstation:~/MyVault/Estuary_LIVE/customer-est-ana$ git push

