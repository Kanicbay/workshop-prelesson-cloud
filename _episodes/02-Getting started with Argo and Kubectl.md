---
title: "Getting started with Argo and Kubectl"
teaching: 15
exercises: 15
questions:
- "How to setup a workflow engine to submit jobs?"
- "How to use Kubectl commands?"
- "What is kubectl?"
- "What is Argo workflows?"
- "What kind of services/resources will I need to instantiate in my cluster?"

objectives:
- "Understand how to run a simple workflows in a commercial cloud environment"
- "Lear what the kubectl command can do"
- "Appreciate the necessity for the Argo workflows tool (or similar)"
- "Lear how to set up different services/resources to get the most of your cluster"

keypoints:
- "kubectl is the ruler of GKE"
- "Argo is a very useful tool for running workflows and parallel jobs"
- "To be able to write, read and extract data, a few services/resources need to be set up on the GCP"
---

## The `kubectl` command

Just as `gcloud` is the *one command to rule them all* for the GCP, the `kubectl` command is the main tool for interacting with your K8s cluster. You will use it to do essentially anything in the cluster. [Here](https://kubernetes.io/docs/reference/kubectl/cheatsheet/) is the official *cheatsheet*, which is very useful but already very long.

Let's run a few examples.

* Get the status of the nodes in your cluster:

```
kubectl get nodes  
```

* Get the cluster info:

```
kubectl cluster-info  # Display addresses of the master and services

```

Let's list some kubernetes components:

* Check pods

```
kubectl get pod
```

* Check the services

```
kubectl get services
```

We don't have much going on.  Let's create some components.

* Inspect the `create` operation

```
kubectl create -h
```
Note there is no `pod` on the list, so in K8s you don't create pods but *deployments*.  These will create pods, which will run under the hood.

* Let's create an application, it does not matter which.  Let's go for `nginx`:

```
kubectl create deployment mynginx-depl --image=nginx
```

The `nginx` image will be pulled down from the Docker Hub.
This is the most minimalist way of creating a deployment.

* Check the deployments

```
kubectl get deployment
```

* Check pods

```
kubectl get pod
```

## Namespaces

Namespaces are a kind of reservations in your K8s cluster.  Let's create one for the Argo workflow we will user

```
kubectl create ns <NAMESPACE>
```

> ## Kubernetes namespaces
>
> The above commands as well as most of the following use a flag `-n argo`,
> which defines the namespace in which the resources are queried or created.
> [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
> separate resources in the cluster, effectively giving you multiple virtual
> clusters within a cluster.
>
> You can change the default namespace to `argo` as follows:
>
> ~~~
> kubectl config set-context --current --namespace=argo
> ~~~
> {: .bash}
>
{: .testimonial}

## Argo as a workflow engine

While jobs can also be run manually, a workflow engine makes defining and
submitting jobs easier. In this tutorial, we use
[argo](https://argoproj.github.io/argo/quick-start/).
Install it into your working environment with the following commands
(all commands to be entered into the cloud shell):

While jobs can also be run manually, a workflow engine makes defining and submitting jobs easier. In this tutorial, we use [argo quick start](https://argoproj.github.io/argo-workflows/quick-start/) page to install it:

```
kubectl create ns argo
kubectl apply -n argo -f https://raw.githubusercontent.com/argoproj/argo-workflows/stable/manifests/quick-start-postgres.yaml
```

You can now check that argo is available with

```bash
argo version
```

We need to apply a small patch to the default argo config. Create a file called
`patch-workflow-controller-configmap.yaml`:

```yaml
data:
  artifactRepository: |
    archiveLogs: false
```

Apply:

```shell
kubectl patch configmap workflow-controller-configmap -n argo --patch "$(cat patch-workflow-controller-configmap.yaml)"
```

Run a simple test flow:

```
argo submit -n argo --watch https://raw.githubusercontent.com/argoproj/argo-workflows/master/examples/hello-world.yaml
argo list -n argo
argo get -n argo @latest
argo logs -n argo @latest
```

{% include links.md %}
