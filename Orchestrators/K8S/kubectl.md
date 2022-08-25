---
title: kubectl
description: 
published: true
date: 2022-08-07T19:59:51.544Z
tags: 
editor: markdown
dateCreated: 2022-08-07T19:59:39.352Z
---

# Kubectl notes

## Basics commands

Get nodes infos : `kuberctl get nodes`
Get pods infos : `kuberctl get pod`
Get services infos : `kuberctl get services`

To create a pod we need to create a deployment using an image.

Lets create an Nginx deployment : `kubectl create deployment nginx-depl --image=nginx`

(Or with a zsh plugin : `kcd nginx-depl --image=nginx`

This will download latest image deployment from docker

Deployment also manage replicaset :

```bash
kubectl get replicaset
NAME                   DESIRED   CURRENT   READY   AGE
nginx-depl-58458fcbd   1         1         1       2m50s
```

## Abstraction layers

*Top to bottom*

**Deployment** manages a 

**ReplicaSet** manages a

**Pod** is an abstration of

**Container**

Everything below a deployment is managed by Kubernetes.

# Managing deployment

Log a deployment : `kubectl logs <POD-NAME>`

Get a sheel inside a container : `kubectl exec -it <POD-NAME> -- bin/bash`

Delete a deployment : `kubectl delete deployment <POD-NAME>` 

Deleting a deployment will delete container, pod, and replicaSet

Apply a config file : `kubectl apply -f filename.yaml`
