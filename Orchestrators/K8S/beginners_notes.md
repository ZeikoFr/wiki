---
title: K8S Beginners notes
description: My personnal notes for K8S has a beginner
published: true
date: 2022-08-02T18:09:47.170Z
tags: orchestrators, k8s
editor: markdown
dateCreated: 2022-07-19T09:50:56.771Z
---

# K8S Beginners notes

## What is K8S

orchestration framework

## Why k8s

trend from monolith to microservices
increased usage of containers
Need of management tool for thousand of containers

## Feature of orchestration

HA, scalability, disaster recovery

# K8S components

## Nodes and pods

Pod : smallest unit of k8s
      abstraction over container
You only interact with kubernete layer

Pod usually run 1 app per Pod
each pod get its own ip address

Pod are ephemeral, new ip address on re-creation

## Service and Ingress

### Service
Service : Permanent ip address

lifecycle of Services and Pods are not connected ie: if pod dies service do not die

So a pod is given a service 

There's external and internal service (example : web service, db service)

### Ingress

Ingress take the request and forward it to the service (like a reverse proxy)

## ConfigMap and Secrets

### ConfigMap

External configuration to your app

Contain config data of app (url to database and othe services)

Then connect configmap to the pod. So when a url change you only change it in the config map

Do not put secrets in ConfigMap

### Secret

Used to store secret data
base64 encoded
The built-in security mechanism is not enabled by default

Use it as environnement variable or as properties files

## Volumes

Attach a physical storage to a pod
Can be local or remote (outside K8S cluster)

K8S doesn't manage data persistance.

## Deployement and statefull state

The point of orchestrator is to have replica to avoid downtime.

So a replica is on another node.

The replica is connected to the same service.

So service has permanent ip and is a load balancer

### Deployement

You define blueprint for a pod (called a deployments)

You define the number of replica in the deployment

DB can't be replicated with deployment

### Statefulset

Service created for statefull app (mysql, mongodb...)

Make sure that DB read/write are synchronised

DB are often outside of k8s cluter

# K8S architecture

## Nodes processes

Each nodes has multiple pods on it

**3 processes must be installed on every node**

- *container runtime* (docker, podman...) 

- The process that schedule the pods and containers is *Kubelet*

Communication between nodes is done via services

- *Kube proxy* forwards the request

## How do you interact with the cluster ?

All managing process are done by MasterNodes

** 4 processes on master nodes**

- *Api Server* Cluster Gateway, update query, act as gatekeeper for auth

Flow : request => Api server => Validation => other process

- *Scheduler* 

Flow : Schedfule new pod => Api Server => Scheduler 

The scheduler will check the ressource needed for the pod, then decide which node will be used

- *Controller manager* Detect state changes in cluster (dead pod and such)

Flow : Controller Manager => scheduler => kubelet

- *etcd* cluster brain, cluster changes get stored in the key value store

**Application data are NOT stored in etcd**

Usually K8S cluster have multiple master with api server load balenced
