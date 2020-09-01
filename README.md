# Sample GitOps Repo

## Introduction

The given sample GitOps repo is an indication of the capability of the GitOps tool, the given repo structure constituting the entire GitOps configuration can be created by running a couple of day-1 and day-2 commands. A high level structure of the GitOps repo constitutes the config, environments folder and the pipelines.yaml file. 

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
The pipelines.yaml is a representatioon of the current directory structure, it throws light on the essential bits of the GitOps repo. The current GitOps repo structure can be broken down based on the pipelines.yaml file.

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

The config directory refers to the special environments which contain the configuraion manifests for the argocd and cicd environments.

```
├── config
    ├── argocd
    │   
    └── demo-cicd
```
* #### Argocd sub-folder

  ArgoCD is used to perform Continuous Deployment of Applications. When an Application is created in the target Environment an ArgoCD application is also created and kept in the ArgoCD Environment. The user is reponsible for creating deployment.yaml in the "config" folder for the application. ArgoCD will deploy the application based on the user-provided deployment specification and re-deploy it automatically when the specification is changed. The argocd directory contains all the necessary resources to perform continous deployment to ensure that the live application state on the cluster is in sync with the target state on the cluster. The pipelines.yaml holds information about the name of the namespace in which the argocd operator as well the argocd resources will be applied.

* #### Cicd sub-folder

  The CI/CD Environment is a special Environment that contains CI/CD pipelines. These pipelines respond to changes in GitOps configuration repository and Application/Service source repositories. They are responsible for keeping the resources in the cluster in-sync with the configurations in Git and re-build/re-deploy application/service images. The pipelines.yaml holds information on the namespace in which the tekton pipelines have been deployed.

### Environments folder

Within a Pipelines Model, there are many Environments which hold Applications and Services. Each Environment has its own namespace/project as defined in openshift. The sample environments directory has the structure listed below.

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

* #### (Plain Old) Environment:

  Within a Pipelines Model, there are many Environments which hold Applications and Services. Each Environment has its own namespace.

* #### Application:
 
  An Application is a logical grouping of Services. It contains references to Services. When an Application is deployed, all referenced Services are deployed. Two Applications can reference to a same Service. Each Application can have specific customization to the Service it references/deploys. A Service is not intendedto be deployed by itself (without an Application).

* #### Service:

  A Service can have a source repository and an image repository. Services are unique within an Environment. However, no two Services can share a same source Git reposiotry even though they belong to different Environments. In the pipelines.yaml file, the services holds information to the type of trigger binding(extracts useful information from payloads) and trigger template(holds information essential for pipeline runs), the sealed secret name and the namespace in which it is present.

Our current pipelines model corresponds to the structure defined in the pipelines.yaml file. We can have as many environments, an environment folder inturn comprises of applications. An application has no real significance without services,hence the service are located within the application folder. A service contains the necessary deployment and config files, services are unique to an environment.

## Disclaimer

This repository cannot be applied to the cluster since the sealed secrets are encrypted with accordance to the cluster at the time of creation of the repository.
To create a GitOps repository that can be applied to the cluster, kindly run the commands showcased below.

## Commands used to build sample repo
The following repo is build by running the following 3 commands:

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
