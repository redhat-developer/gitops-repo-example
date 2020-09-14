# Sample GitOps Repo

## Introduction

The given sample GitOps repo is an indication of the capability of the GitOps tool, the given repo structure constituting the entire GitOps configuration can be created by running a couple of day-1 and day-2 commands. A high level structure of the GitOps repo constitutes the config, environments folder and the pipelines.yaml file. 

#### Important Note:

This repository cannot be applied to the cluster since the sealed secrets are encrypted with the private key in the cluster at the time the repository was created.

### Pipelines.yaml

```yaml
config:
  argocd:
    namespace: argocd
  pipelines:
    name: demo-cicd
environments:
- apps:
  - name: app-taxi
    services:
    - name: taxi
      pipelines:
        integration:
          bindings:
          - demo-dev-app-taxi-taxi-binding
          - github-push-binding
      source_url: https://github.com/ishitasequeira/taxi.git
      webhook:
        secret:
          name: webhook-secret-demo-dev-taxi
          namespace: demo-cicd
  name: demo-dev
  pipelines:
    integration:
      bindings:
      - github-push-binding
      template: app-ci-template
- apps:
  - name: bus-app
    services:
    - name: bus
      source_url: https://github.com/ishitasequeira/bus.git
      webhook:
        secret:
          name: webhook-secret-demo-stage-bus
          namespace: demo-cicd
    - name: redis
      webhook:
        secret:
          name: webhook-secret-demo-stage-redis
          namespace: demo-cicd
    - name: bus1
      source_url: https://github.com/ishitasequeira/bus1.git
      webhook:
        secret:
          name: webhook-secret-demo-stage-bus1
          namespace: demo-cicd
  - name: truck-app
    services:
    - name: truck
      source_url: https://github.com/ishitasequeira/truck.git
      webhook:
        secret:
          name: webhook-secret-demo-stage-truck
          namespace: demo-cicd
    - name: truck-frontend
      source_url: https://github.com/ishitasequeira/truck-frontend.git
      webhook:
        secret:
          name: webhook-secret-demo-stage-truck-frontend
          namespace: demo-cicd
  name: demo-stage
- apps:
  - name: bus-app
    services:
    - name: car
      source_url: https://github.com/ishitasequeira/car.git
      webhook:
        secret:
          name: webhook-secret-prod-car
          namespace: demo-cicd
    - name: cycle
      source_url: https://github.com/ishitasequeira/cycle.git
      webhook:
        secret:
          name: webhook-secret-prod-cycle
          namespace: demo-cicd
    - name: rdisc-cli
      webhook:
        secret:
          name: webhook-secret-prod-rdisc-cli
          namespace: demo-cicd
  name: prod
  pipelines:
    integration:
      bindings:
      - github-push-binding
      template: app-ci-template
gitops_url: https://github.com/rhd-gitops-example/gitops.git
```
The pipelines.yaml is a representation of the current directory structure, it throws light on the essential bits of the GitOps repo. The current GitOps repo structure can be broken down based on the pipelines.yaml file.

### High level directory structure

```
.
├── config
│   ├── argocd
│   │   
│   └── demo-cicd
│       ├── base
│       │   ├── 01-namespaces
│       │   ├── 02-rolebindings
│       │   ├── 03-secrets
│       │   ├── 04-tasks
│       │   ├── 05-pipelines
│       │   ├── 06-bindings
│       │   ├── 07-templates
│       │   ├── 08-eventlisteners
│       │   ├── 09-routes
│       │   ├── 10-commit-status-tracker
│       │   └── kustomization.yaml
│       └── overlays
│           └── kustomization.yaml
├── environments
│   └── prod
│       ├── apps
│       │   └── bus-app
│       │       └── services
│       │           ├── car
│       │           ├── cycle
│       │           └── rdisc-cli
│       └── env
└── pipelines.yaml
```

The entire structure can be explained by taking a look at the pipelines.yaml file since it contains a high level view of the contents in the GitOps repo. At the topmost level is divided into the config and environments directory.

### Config folder

The config directory refers to the special environments which contain the configuraion manifests for the ArgoCd and CI/CD environments.

```
├── config
    ├── argocd
    │   
    └── demo-cicd
```
* #### [ArgoCd sub-folder](https://github.com/rhd-gitops-example/docs/tree/master/model#argocd-environment)

* #### [CI/CD sub-folder](https://github.com/rhd-gitops-example/docs/tree/master/model#cicd-environment)

### Environments folder

Within a pipelines model, there are many environments which hold applications and services. Each environment has its own namespace/project as defined in openshift. The sample environments directory has the structure listed below.

```
├── environments
   └── prod
        ├── apps
        │   └── bus-app
        │       └── services
        │           ├── car
        │           ├── cycle
        │           └── rdisc-cli
        └── env

```

It comprises of :

* #### [Environment](https://github.com/rhd-gitops-example/docs/tree/master/model#plain-old-enviroment)

* #### [Application](https://github.com/rhd-gitops-example/docs/tree/master/model#application)

* #### [Service](https://github.com/rhd-gitops-example/docs/tree/master/model#service)

## Commands used to build sample repo
To create a GitOps repository that can be applied to the cluster, kindly run the commands showcased below. The following repo is build by running the following 3 commands:

### Day-1 operation
```shell
$ odo pipelines bootstrap --service-repo-url https://github.com/ishitasequeira/taxi.git \
  --dockercfgjson ~/Downloads/isequeir-robot-auth.json \
  --gitops-repo-url https://github.com/ishitasequeira/gitops166.git \
  --image-repo quay.io/isequeir/taxi --prefix demo --output ~/go/src/github.com/openshift/gitops \
  --sealed-secrets-svc sealed-secrets-controller --sealed-secrets-ns kube-system 
```
### Day-2 operation
```shell
$ odo pipelines environment add   --env-name prod \
  --pipelines-file  ~/go/src/github.com/openshift/gitops/pipelines.yaml
```

```shell
$ odo pipelines service add  --app-name bus-app --env-name prod \
  --pipelines-file ~/go/src/github.com/openshift/gitops \
  --service-name car --sealed-secrets-ns kube-system \
  --sealed-secrets-svc sealed-secrets-controller \
  --git-repo-url https://github.com/ishitasequeira/car.git 
```
