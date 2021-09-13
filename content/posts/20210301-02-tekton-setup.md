---
date: "2021-03-01"
categories:
  - ops
tags:
  - tekton
title: Pipelines com o Tekton - Instalação
draft: true
---

## Deploy Cert Manager

```shell
kubectl create ns cert-manager
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install \
 cert-manager jetstack/cert-manager \
 -n cert-manager \
 --version v1.1.0 \
 --set installCRDs=true
```

```bash
openssl genrsa -out ca.key 2048
```

```bash
openssl req -x509 -new -nodes -key ca.key -subj "/CN=integr8.me" -days 3650 -reqexts v3_req -extensions v3_ca -out ca.crt
```

```bash
kubectl create secret tls integr8-ca-key-pair \
 -n cert-manager \
 --cert=ca.crt \
 --key=ca.key \
```

```yml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: integr8me
  namespace: sandbox
spec:
  ca:
    secretName: integr8-ca-key-pair
    crlDistributionPoints:
      - "https://integr8.me"
```

```bash
kubectl apply \
 -n cert-manager \
 -f cert-manager-cluster-issuer.yml
```

``` bash
kubectl create ns tekton-pipelines
kubectl apply \
 -n tekton-pipelines \
 -f https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml \
 -f https://storage.googleapis.com/tekton-releases/triggers/latest/release.yaml \
 -f https://github.com/tektoncd/dashboard/releases/latest/download/tekton-dashboard-release.yaml \
 -f tekton-ingress-dashboard.yml
```

```
kubectl create ns tekton-builds
kubectl apply -n tekton-builds -f rbac-and-account.yml
```

```yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tekton-triggers-sa
secrets:
  - name: gh-token
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: tekton-triggers-roles
rules:
  - apiGroups: ["triggers.tekton.dev"]
    resources:
      ["eventlisteners", "triggerbindings", "triggertemplates", "triggers"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["configmaps", "secrets"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["tekton.dev"]
    resources: ["pipelineruns", "pipelineresources", "taskruns"]
    verbs: ["create"]
  - apiGroups: [""]
    resources: ["serviceaccounts"]
    verbs: ["impersonate"]
  - apiGroups: ["policy"]
    resources: ["podsecuritypolicies"]
    resourceNames: ["tekton-triggers"]
    verbs: ["use"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: tekton-triggers-binding
subjects:
  - kind: ServiceAccount
    name: tekton-triggers-sa
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: tekton-triggers-roles
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: tekton-triggers-clusterrole
rules:
  - apiGroups: ["triggers.tekton.dev"]
    resources: ["clustertriggerbindings"]
    verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tekton-triggers-clusterrolebinding
subjects:
  - kind: ServiceAccount
    name: tekton-triggers-sa
    namespace: tekton-builds
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: tekton-triggers-clusterrole
```
