---
layout: default
title: Kubernetes
description: Useful for using Kubernetes
tags: kubernetes devops coding
---

## Kubernetes components and terms

+ CRD - Custom Resource Definition
+ CRI - Container Runtime Interface (containerd, cri-o, dockershim)
+ CNI - Container Network Interface (AWS, Google and other provider implementations; calico, flannel, cilium)
+ CSI - Container Storage Interface has [different drivers](https://github.com/kubernetes-csi) that allow seamless certificate and secret insertion.
+ HPA - Horizontal Pod Autoscaler (activate in a deployment)
+ Policies - [OPA Gatekeeper](https://github.com/open-policy-agent/gatekeeper), [Kubewarden](https://www.kubewarden.io/)
+ kubelet - agent running on each worker node
+ cilium CNI uses eBPF directly, allowing for API aware network policies at layer 7. It also has transparent wireguard traffic encryption.

### Kubernetes workloads

+ Deployment - declarative changes for Pods and ReplicaSets, no state
+ StatefulSet - deployment of pods with state, with guarantees about the ordering and uniqueness of pods.
+ DaemonSet - defines pods that provide node-local facilities.
+ Job, CronJob - defines a task that runs to completion. NB! 3rd party CRD ecosystem

## [Structure of a Kubernetes cluster](https://kubernetes.io/docs/concepts/architecture/)

### Control Plane

+ Controllers help match the actual state to desired state
+ CCM Cloud Controller Manager - for resources outside Kubernetes (load balancer etc) via the cloud API
+ CM Controller Manager - builtin controllers like Job -, Deployment Controller
+ etcd - higly available key-value store
+ scheduler - assigns pods to new nodes based on usages
+ api directs calls to other components

### Data Plane - contains worker node(s) which contain pods

+ kubelet - spawns, probes (monitors) workloads (that contain pods)
+ pod - smallest deployable unit that can be managed, a set of containers. Init container, application container, ephemeral (debugging)

## [Objects in Kubernetes](https://kubernetes.io/docs/concepts/overview/working-with-objects/)

### Namespaces

Namespaces help different projects, teams, or customers to share a Kubernetes cluster by providing:

+ A scope for Names.
+ A mechanism to attach authorization and policy to a subsection of the cluster.

```sh
kubectl config set-context dev --namespace=development \
  --cluster=super_kubernetes \
  --user=super_kubernetes
kubectl config set-context prod --namespace=production \
  --cluster=super_kubernetes \
  --user=super_kubernetes

kubectl config view
```
```sh
kubectl config use-context dev
kubectl config current-context
```

### [Labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)

Labels are key-value pairs that help filtering (API). They are intended to be used to specify identifying attributes relevant to users.
Annotations (key-value pairs) have no validation and are meant for notes, nonidentifying metadata.
