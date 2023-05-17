## Helm Basic Commands

|Command|Description|
|--|--|
|`helm repo add`| add a chart repository|
|`helm repo add`|add a chart repository|
|`helm search <repo/hub> <chart>`|seach for a chart in repo or hub|
|`helm install <chart>`|install all kubernetes objects defined in chart to the cluster the kubeconf points to |
|`helm install <chart> dry-run`|dry-run Installation|
|`helm list`|list all the releases|
|`helm upgrade <chart>`|upgrade a release|
|`helm rollback <chart>`|rollback a release|
|`helm history`|list history|
|`helm create`|create a new chart directory
|`help package`|create a chart archive from chart directory|


## Helm charts

|Command|Description|
|--|--|
|`helm repo list`|list the repos installed|
|`helm repo add stable https://charts.helm.sh/stable`|add repo|
|`helm repo add bitnami https://charts.bitnami.com/bitnami`|add repo|
|`helm search repo bitnami/mysql`|search for chart|
|`helm show chart bitnami/mysql`|examine the chart|
|`helm show readme bitnami/mysql`|show the readme|
|`helm show values bitnami/mysql`|show the values|
|`helm install <release_name> <repo_name>/<chart_name>`|Install a chart|
|`helm install mysql bitnami/mysql`|install sql chart|
|`helm install mysql bitnami/mysql --dry-run`|Install using "dry-run" to see what is done|
|`helm install mysql bitnami/mysql --dry-run --debug`|dry-run with debug and more info|
|`helm history mysql`|Check the history|
|`helm list --all`|list the releases|
|`helm uninstall mysql --keep-history`|uninstall the chart|
|`helm upgrade mysql bitnami/mysql --version 9.3.4`|upgrade the chart|
|`helm pull bitnami/mysql --untar`| pull a chart locally to explore [image-link](helm_chart_structure.png)|


>NOTE
>After each release, the `revision number` gets updated. Even after the upgrade
>Changes can be noticed using the `list` & `history` commands


## Creating and Managing Helm Charts & Local Repos

### Creating & Publishing Local Charts

|Command|Description|
|--|--|
|`helm create <chart_name>`|Create a chart with default dir structure|
|`helm install <release_name> <chart_path>`| deploy the created chart|
|`helm package <chart_path> --destination <target_path>`|package the chart for pushing to repository|


```bash
dhayanand@master1:/tmp/dump$ helm create mychart
Creating mychart
dhayanand@master1:/tmp/dump$
dhayanand@master1:/tmp/dump$ tree -L 2 mychart
mychart
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
└── values.yaml

3 directories, 9 files
dhayanand@master1:/tmp/dump$ 
```

### Using ChartMuseum to create a local repo

|Command|Description|
|--|--|
|`helm repo add chartmuseum https://chartmuseum.github.io/charts`| install the repo|
|`helm install chartmuseum chartmuseum/chartmuseum --set env.open.DISABLE_API=false`|deploy ChartMuseum|
|Get the Pod name : `export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=chartmuseum" -l "app.kubernetes.io/instance=chartmuseum" -o jsonpath="{.items[0].metadata.name}")` <br> Expose the port : `kubectl port-forward $POD_NAME 8080:8080 --namespace default`|portforward the service|
|`helm repo add <repo_name> http://127.0.0.1:8080`| add the local repo|
|`curl --data-binary "@${PACKAGE_NAME}-${PACKAGE_VERSION}.tgz" http://127.0.0.1:8080/api/charts`|upload chart to repo|




#### Snippet of local repo usage

```bash
dhayanand@master1:/tmp/dump$ helm repo add chartmuseum https://chartmuseum.github.io/charts
"chartmuseum" has been added to your repositories
dhayanand@master1:/tmp/dump$ helm install chartmuseum chartmuseum/chartmuseum --set env.open.DISABLE_API=false
NAME: chartmuseum
LAST DEPLOYED: Tue Nov  8 19:02:44 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

Get the ChartMuseum URL by running:

  export POD_NAME=$(kubectl get pods --namespace default -l "app=chartmuseum" -l "release=chartmuseum" -o jsonpath="{.items[0].metadata.name}")
  echo http://127.0.0.1:8080/
  kubectl port-forward $POD_NAME 8080:8080 --namespace default
dhayanand@master1:/tmp/dump$
dhayanand@master1:/tmp/dump$ export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=chartmuseum" -l "app.kubernetes.io/instance=chartmuseum" -o jsonpath="{.items[0].metadata.name}")
dhayanand@master1:/tmp/dump$
dhayanand@master1:/tmp/dump$ helm repo add http://127.0.0.1:8080
"localrepo" has been added to your repositories
dhayanand@master1:/tmp/dump$
dhayanand@master1:/tmp/dump$ helm repo list
NAME                    URL                                               
kubernetes-dashboard    https://kubernetes.github.io/dashboard/           
prometheus-community    https://prometheus-community.github.io/helm-charts
bitnami                 https://charts.bitnami.com/bitnami                
chartmuseum             https://chartmuseum.github.io/charts              
localrepo               http://127.0.0.1:8080                                                         
dhayanand@master1:/tmp/dump$
dhayanand@master1:/tmp/dump$ curl --data-binary @mychart-0.1.0.tgz http://127.0.0.1:8080/api/charts
{"saved":true}
dhayanand@master1:/tmp/dump$
dhayanand@master1:/tmp/dump$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "localrepo" chart repository
...Successfully got an update from the "kubernetes-dashboard" chart repository
...Successfully got an update from the "chartmuseum" chart repository
...Successfully got an update from the "prometheus-community" chart repository
...Successfully got an update from the "bitnami" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈
dhayanand@master1:/tmp/dump$
dhayanand@master1:/tmp/dump$ helm search repo localrepo/mychart
NAME CHART VERSION APP VERSION DESCRIPTION
localrepo/mychart 0.1.0 1.16.0 A Helm chart for Kubernetes
dhayanand@master1:/tmp/dump$
dhayanand@master1:/tmp/dump$ helm list
NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
chartmuseum             default         1               2022-11-08 20:38:55.716499817 +0000 UTC deployed        chartmuseum-3.9.1               0.15.0     
kubernetes-dashboard    default         1               2022-10-12 11:38:16.1986895 +0000 UTC   deployed        kubernetes-dashboard-5.11.0     2.7.0      
mychart                 default         2               2022-11-08 19:33:13.944719351 +0000 UTC deployed        mychart-0.1.0                   1.16.0     
dhayanand@master1:/tmp/dump$ 
dhayanand@master1:/tmp/dump$ helm delete mychart
release "mychart" uninstalled
dhayanand@master1:/tmp/dump$ helm list
NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
chartmuseum             default         1               2022-11-08 20:38:55.716499817 +0000 UTC deployed        chartmuseum-3.9.1               0.15.0     
kubernetes-dashboard    default         1               2022-10-12 11:38:16.1986895 +0000 UTC   deployed        kubernetes-dashboard-5.11.0     2.7.0      
dhayanand@master1:/tmp/dump$ helm install mychart localrepo/mychart
NAME: mychart
LAST DEPLOYED: Tue Nov  8 21:10:03 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
A test Helm Chart
dhayanand@master1:/tmp/dump$ helm list
NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
chartmuseum             default         1               2022-11-08 20:38:55.716499817 +0000 UTC deployed        chartmuseum-3.9.1               0.15.0     
kubernetes-dashboard    default         1               2022-10-12 11:38:16.1986895 +0000 UTC   deployed        kubernetes-dashboard-5.11.0     2.7.0      
mychart                 default         1               2022-11-08 21:10:03.616441533 +0000 UTC deployed        mychart-0.1.0                   1.16.0     
dhayanand@master1:/tmp/dump$ 
```

### Remote Repo - GitHub

- copy the helm package to the git repo dir
- generate the `index.yaml` file that will have all the information about our package chart
  `helm repo index .`
- push the code / repo to GitHub
- add GitHub as the Helm repo using the `raw` url of the repo as below
  `helm repo add <repo_name> https://raw.githubusercontent.com/<user_name>/<repo_name>/<branch_name>`
