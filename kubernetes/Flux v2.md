# Flux Basics

Flux is one of the application used to implement GitOps practices.

Consists of multiple controllers for different purposes

**Source-Controller:**
- responsible for retrieving the desired state from a suitable source (VCS, s3 storage bucket or Helm repo)
- periodically fetches the content for use within the cluster

**Kustomize-Controller:**
- responsible for reconciling the differences between actual & desired state of the configuration

**Helm-Controller:**
- similar to Kustomize-Controller but works for Helm charts

**Notification-Controller**
- responsible for receiving events from systems outside the cluster (a webhook from the GitHub for example) that can be consumed by the GitOps toolkit components
-  also responsible for disseminating alerts and events from the GitOps components to external systems for our consumptions (slack, teams etc.,)

Additional toolkit components:

**Image-Reflector Controller**
- monitors specific repos of given container registries and fetches metadata associated with new images

**Image Automation Controller**
- takes care of writing the latest available tag to the relevant YAML manifest which is committed and pushed back to source

# Cheat Sheet

```make

```

## Bootstrap Flux

```bash
# precheck
flux check --pre

# to uninstall
flux uninstall

# Setup Git PAT
export GITHUB_TOKEN=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

# install will only install the components
# bootstrap will install components and save the configurations is a VCS or s3 bucket
# bootstarp Flux with GitHub
flux bootstrap github \
  --owner=dhayanands \
  --repository=flux2-demo \
  --personal true \
  --components-extra=image-reflector-controller,image-automation-controller
```

## Configure Notification Provider and Alerts

```bash
# clone the GitHub repo and perform the below inside the dir

# secret file
cat ../teams-webhook-url
address==https://pangearocks.webhook.office.com/webhookb2/YOUR/TEAMS/WEBHOOK
# create secret
kubectl create secret generic teams-url \
--from-literal=https://pangearocks.webhook.office.com/webhookb2/6bce4f03-726f-428a-abbd-89ad3e9cb83e@e0ef37b5-3c94-4d42-8e5d-20992b7ea1c4/IncomingWebhook/3328e422f54b46c6abcb93822a5f5474/abe1118a-27cc-4bb5-a51f-3cfe484cff91

--from-literal=address=https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK

# create seperate directories for provider & alert definition
mkdir -p notifications/{provider,alerts}

# generate the definitions using flux command
flux create alert-provider msteams \
	--type=msteams \
	--channel=SRE_DEMO \
	--secret-ref=teams \
	--username=FluxBot \
	--namespace=default \
	--export > notifications/providers/msteams-provider.yaml

flux create alert msteams-bot-alert\
  --event-severity=info \
  --event-source="GitRepository/*,Kustomization/*" \
  --provider-ref=msteams \
  --namespace=default \
  --export > notifications/alerts/msteams-bot-alert.yaml

# Create kustomization file
## not generatlly needed as kustomize will apply all manifests it finds in the source but it isa good practice to create one
cd notifications
kustomize create --autodetect --recursive

# push the files to GitHub repo
git add notifications
git commit -m ""
git push

# validate the if the resources are created
kubectl get providers.notification.toolkit.fluxcd.io -A
kubectl get alerts.notification.toolkit.fluxcd.io -A
```

Workflow:

Application Repo --CICD--> Update tags in Deployment Repo ----> FLUX2 which is monitoring the deploy repo -> pulls the configuration to cluster and applies it using controllers

Establish ssh trust between Flux & GitHUb

```bash
# generate ssh key pair
flux create secret git gitops-kind-demo-auth \
    --url=ssh://github.com/dhayanands/kind-cluster-demo \
    --namespace=default

# check the secret
kubectl get secrets gitops-kind-demo-auth -o jsonpath='{.data}'
kubectl get secrets gitops-with-flux -o yaml | yq '.data'

# add the public key to github deploy repo

```

Create custom resource for deployment

```bash
mkdir flux2-demo/{sources,kustomizations}

flux create source git nginxhello \
    --url=ssh://github.com/dhayanands/kind-cluster-demo \
    --branch=main \
    --secret-ref=gitops-kind-demo-auth \
    --namespace=default \
    --export > flux2-demo/sources/nginxhello-source.yaml

flux create kustomization nginxhello \
    --source=GitRepository/nginxhello.default \
    --path=./deploy \
    --prune=true \
    --target-namespace=default \
    --namespace=default \
    --export > flux2-demo/kustomizations/nginxhello-kustomization.yaml

cd flux2-demo/sources
kustomize create --autodetect
cd flux2-demo/kustomizations
kustomize create --autodetect


kubectl get gitrepositories
kubectl get kustomizations

```


Create deployment under the deployment repo

```bash
cd kind-cluster-demo
mkdir deploy
vim deploy/nginx.yaml
git add .
git commit -m "added nginx deployment under deploy dir"
git push
```
