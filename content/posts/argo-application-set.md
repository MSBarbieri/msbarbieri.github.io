---
title: Automate Delivery specification with ArgoCD ApplicationSet
description:
toc: true
authors: []
tags: []
categories: [GitOps, Automation, CD]
series: []
date: 2022-05-25T16:39:45-03:00
lastmod: 2022-05-25T16:39:45-03:00
featuredVideo:
featuredImage:
draft: false
---

I'm without motivation enough to write a post talking about how a delivery pipeline should be, how to relate real-state x desired-state or why you should use this or that, i particularly prefer ArgoCD by a simple resource called ApplicationSet. 


ArgoCD's ApplicationSet is a awesome feature to a delivery pipeline.

<b> it make you specify in a manifest how and where k8s manifests should be applied</b>
the argocd read the applications sets, and create a pipeline which will apply source from parameters into destinations

You have a lot of features to that and i strongly recommend you to look on the [DOC](https://argocd-applicationset.readthedocs.io/en/stable/), but here i will show you how to use the basic, and how i like to use.

The basic structure to an ApplicationSet:

``` yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: <RESOURCE_NAME>
spec:
  generators:
  - list:
      elements:
      - cluster: <CLUSTER_NAME> # variable name could be local, dev prod
        url: <CLLUSTER_URL> # https://kubernetes.default.svc
        namespace: <NAMESPACE> # another variable
        # <VAR_KEY>: <VAR_VALUE> any var you want for the generate application-set
  template:
    metadata:
      name: <APPLICATION_NAME> # the name of application applied will be local-appllications
      labels:
        <LABEL_NAME>: <LABEL_VALUE>
    spec:
      project: default
      destination:
        server: '{{url}}' # the cluster you want ArgoCD apply the manifest
        namespace: '{{namespace}}' # the namespace you want this to be applied
      source: # an example of source in a specific repo
        repoURL: <REPOSITORY URL>.git
        targetRevision: <BRANCH> # default in HEAD
        path: <PATH_OF_KUBERNETES_MANIFEST>
```

the argocd is capable to read directly from a repository or a helm chart, and is capable of read different types of inputs, like a classic kubernetes resource, kustomize and others.

> :information_source: one of my favorite tricks is to point to a helm path in a github repository, with this you will know automatically if a new version of 

``` yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: sealed-secrets-helm-chart
spec:
  generators:
  - list:
      elements:
      - cluster: local
        url: https://kubernetes.default.svc
        namespace: sealed-secrets
  template:
    metadata:
      name: 'sealed-secrets-{{cluster}}
      labels: 
        type: "security"
        cluster: "{{cluster}}"
    spec:
      project: default
      destination:
        server: '{{url}}'
        namespace: '{{namespace}}'
      source:
        repoURL: https://github.com/bitnami-labs/sealed-secrets
        targetRevision: HEAD
        path: helm/sealed-secrets
      syncPolicy:
        syncOptions:
          - CreateNamespace=true
```
this example will install <b>Sealed Secrets</b> using helm structure but using git repo.


And you can construct an repo with multiple application-sets to be installed and create an application-set to check if has a new application-sets to be added. 

Example:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: application-set
spec:
  generators:
  - list:
      elements:
      - cluster: local
        url: https://kubernetes.default.svc
        namespace: argocd
  template:
    metadata:
      name: '{{cluster}}-applications'
      labels:
        general: '{{cluster}}'
        all: 'argocd'
    spec:
      project: default
      destination:
        server: '{{url}}'
        namespace: '{{namespace}}'
      source:
        repoURL: https://github.com/MSBarbieri/Argocd-Application-Sets.git
        targetRevision: HEAD
        path: .
      syncPolicy:
        automated:
          prune: true
          allowEmpty: true
          selfHeal: true
        retry:
          limit: 2
          backoff: 
            duration: 5s
            factor: 2
            maxDuration: 3m0s
```


ArgoCD is one of the most powerful and extendable delivery pipeline application i have founded until today, and you should at test.
