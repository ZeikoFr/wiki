# Minikube notes

## What is Minikube

Usual K8S prod setup

  - Multiple master and worker nodes
  - Separate virtual or physical machine

To test on local machine it is hard to create a full cluster on a single workstation

Minikube allow us to run master and node processes on one machine.

Each node has a docker runtime pre-installed

Usefull for testing on local.

## What is Kubectl

Used to communication with the k8s API server (located on the master).

# Install minikube and kubectl

On arch machine, simply do a `pamac install minikube`

I'm using a vmware virtualization at time of writing so I also need the driver 

`sudo pamac install docker-machine-driver-vmware`

# Use minikube 

Getting the status to see if everything is correct :

```
minikube status 
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

Control Plane = Master


