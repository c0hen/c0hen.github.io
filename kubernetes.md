---
layout: default
title: Kubernetes
description: Useful for using Kubernetes
tags: kubernetes devops coding
---

* Table of contents
{:toc}

## Kubernetes components and terms

+ CRD - Custom Resource Definition
+ CRI - Container Runtime Interface (containerd, cri-o, dockershim)
+ CNI - Container Network Interface (AWS, Google and other provider implementations; calico, flannel, cilium)
+ CSI - Container Storage Interface has [different drivers](https://github.com/kubernetes-csi) that allow seamless certificate and secret insertion.
+ HPA - Horizontal Pod Autoscaler (activate in a deployment)
+ Policies - [OPA Gatekeeper](https://github.com/open-policy-agent/gatekeeper), [Kubewarden](https://www.kubewarden.io/)
+ kubelet - agent running on each worker node
+ manifest files - describe the desired state in terms of Kubernetes API objects.
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

For the apiVersion: key in manifests, print the supported API versions on the server:

```sh
kubectl api-versions
```

KYAML aims to be a safer and less ambiguous YAML subset compatible with existing tooling (alpha available since kubernetes 1.34).

### More ergonomic environment

#### [autocomplete](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#enable-shell-autocompletion) for kubectl (bash)

```sh
source <(kubectl completion bash)
alias k=kubectl
complete -o default -F __start_kubectl k
```
#### kubectl [convert](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-convert-plugin) plugin

Convert manifests between different API versions.

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

namespace-dev.yaml
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
  labels:
    name: development
```
namespace-prod.yaml
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    name: production
```

```sh
kubectl config use-context dev
kubectl config current-context
```

Show help for a subcommand:
```sh
kubectl config --help
```
Apply manifest:

```sh
kubectl apply -f namespace-dev.yaml
```

[kubectx](https://github.com/ahmetb/kubectx) - switch contexts (clusters) faster
kubens - switch namespaces faster

### [Labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)

Labels are key-value pairs that help filtering (API). They are intended to be used to specify identifying attributes relevant to users.  
Annotations are key-value pairs that have no validation and are meant for notes, nonidentifying metadata.

### Finalizers

[Finalizers](https://kubernetes.io/docs/concepts/overview/working-with-objects/finalizers/) are namespaced keys that tell Kubernetes to wait until specific conditions are met before it fully deletes resources that are marked for deletion. Finalizers can [get in the way](https://kubernetes.io/blog/2021/05/14/using-finalizers-to-control-deletion/) of deleting resources in Kubernetes, especially when there are parent-child relationships between objects.
When you create a resource using a manifest file, you can specify finalizers in the
```sh
metadata.finalizers
```
field. When you attempt to delete the resource, the API server handling the delete request notices the values and does the following:

+ Modifies the object to add a metadata.deletionTimestamp field with the time you started the deletion.
+ Prevents the object from being removed until all items are removed from its metadata.finalizers field
+ Returns a 202 status code (HTTP "Accepted")

The relevant controller sees the deletionTimestamp and attempts to satisfy the requirement. On success the controller removes that key from the finalizers field. When the finalizers field is emptied, an object with a deletionTimestamp field set is automatically deleted.

## Kubernetes related projects

#### [Cloud Native Computing Foundation landscape](https://landscape.cncf.io/)
#### [Helm](https://helm.sh/docs/chart_best_practices/conventions) installs charts (packages) into Kubernetes, creating a new release for each installation. To find new charts, you can search Helm chart repositories.
#### [kind](https://kind.sigs.k8s.io/) - for testing local Kubernetes clusters using Docker container 'nodes'.

```sh
kind create cluster # blocks until the control plane reaches a ready status
cat ~/.kube/config
kubectx
kind delete cluster
```
#### [Kubernetes dashboard](https://github.com/kubernetes/dashboard)

+ deploy containerized applications to a Kubernetes cluster
+ troubleshoot your containerized application
+ manage the cluster resources
+ get an overview of applications running on your cluster
+ creating or modifying individual Kubernetes resources (such as Deployments, Jobs, DaemonSets, etc)

#### [minikube](https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fdebian+package) local Kubernetes for learning and development

```sh
minikube start
```
Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```sh
minikube pause # pause minikube without affecting deployed applications
minikube stop
minikube dashboard # opens the local url in your browser when ready
minikube addons list
kubectl cluster-info dump | less
```

#### [chainguard](https://images.chainguard.dev/)

Aims to minimize vulnerabilities by providing container images. Latest versions are free.

#### [checkov](https://www.checkov.io/7.Scan%20Examples/Kubernetes.html)

Evaluation of policies on your Kubernetes files, manifest scanning.

#### [jaeger](https://www.jaegertracing.io/docs/) flexible distributed tracing platform

Deployable as a single binary that can be configured to perform different roles within the Jaeger architecture.

#### [linkerd](https://linkerd.io/2.19/features/)

## Security

### some checks to keep in mind

+ What's running in the container
    + Your Code (Static Analysis)
    + Other peoples code (Dependency Scanning)
    + What's in the image (Image Scanning)
    + Can you scan it while it's running (DAST/RASP/IAST)
+ How is that container going to run
    + Manifest Scanning
    + Admission Control
    + Both of these to check for things like:
    + does it run in a privileged way
    + Is it going to run in the right namespace
    + does it have right guard rails quota/pdb/memory requests
+ How are things going to communicate
    + Network Policies (CNI Provider)
    + Authorization (IdP - identity provider)
    + Encryption (Service Mesh?)
    + Can you protect it at the edge (Web Application Firewall)
+ How does your cluster operate
    + Who has access (RBAC tooling)
    + How are secrets accessed or updated (Secret Management)
+ What is your cluster running on
    + Is your Cloud provider or on-prem hypervisor environment secure?
    + Is the VM/OS underneath secure

### Service Mesh

Run a bunch of networking concerns/features inside a sidecar that each service can use. Instead of implementing them directly in each service.

The usefulness of this scales with the amount of services you have. If you have just 2 services talking to each other, implement the features you care about in the services directly. Decision to use service meshes needs to be made at the beginning of the project, is the extra complexity worth the benefits? Consider capacity planning (increased resource usage), networking design, qualified people etc.

[cilium](https://docs.cilium.io/en/latest/network/servicemesh/index.html)
[istio](https://istio.io/latest/docs/overview/what-is-istio/) - [bookinfo application sample](https://istio.io/latest/docs/examples/bookinfo/)

Some of the things service meshes typically do out of the box (not all of them are equal):
+ Transparent mTLS (mutual TLS) across all services. Very hard to co-ordinate/maintain and get right across 10+ services if done the traditional way.
+ Ingress permissions, service X is allowed to call service Y.
+ Network tracing/observability.
+ Routing. Route all requests to service X at this path to a particular DC/region/version.

Service meshes can also do traffic shifting and steering, retries with back-offs, [canary](https://kubernetes.io/docs/concepts/workloads/management/#canary-deployments), and A/B testing using a rather simple approach. Instead of implementing these things in the application code (especially retries) or using some pieces of your infrastructure to perform canary or traffic shifting, you can define your policies in a YAML file and send it to the MESH control plane which will program the proxies running alongside your workloads. Very important other members of the project are familiar with mesh capabilities!

Service Discovery - a tool like Istio uses Kubernetes Service for Service Discovery. Since each pod has a little proxy running alongside it (called sidecar), each pod in the Mesh is aware of all the other pods IP’s. When two pods talk to each other, your application will use the Service you created to find the server it wants to reach, but traffic going out of your application container is intercepted by the proxy sidecar, policies are applied to it and traffic is sent directly to the receiving end using the pod IP, there is no Load Balancing.

[Ambient service mesh](https://istio.io/latest/blog/2024/ambient-reaches-beta/), without a sidecar. Since istio doesn't use eBPF it could be useful to [integrate it with cilium](https://docs.cilium.io/en/latest/network/servicemesh/istio/).
