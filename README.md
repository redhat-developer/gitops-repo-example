# Mock GitOps Repo

The following repo is build by running the following 3 commands:

```shell
$ odo pipelines bootstrap --service-repo-url https://github.com/ishitasequeira/taxi.git \
  --dockercfgjson ~/Downloads/isequeir-robot-auth.json \
  --gitops-repo-url https://github.com/ishitasequeira/gitops166.git \
  --image-repo quay.io/isequeir/taxi --prefix demo --output ~/go/src/github.com/openshift/gitops \
  --sealed-secrets-svc sealed-secrets-controller --sealed-secrets-ns kube-system 
```

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
