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

## Get access to Google Cloud Platform

Usually, you would need to create your own account (and add a credit card) in
order to be able to access the public cloud. For this workshop, however, you
will have access to Google Cloud via the
[ARCHIVER project](https://www.archiver-project.eu/).
Alternatively, you can create a Google Cloud account and you will get free
credits worth $300, valid for 90 days (requires a credit card).

The following steps will be performed using the
[Google Cloud Platform console](https://console.cloud.google.com/).

> ## Use an incognito/private browser window
>
> We strongly recommend you use an incognito/private browser window
> to avoid possible confusion and mixups with personal Google accounts
> you might have.
>
{: .callout}

## Find your cluster

By now, you should already have created your first Kubernetes cluster.
In case you do not find it, use the "Search products and resources" field
to search and select "Kubernetes Engine". It should show up there.

## Open the working environment

You can work from the cloud shell, which opens from an icon in the tool bar,
indicated by the red arrow in the figure below:

<kbd>
<img src="../fig/startcloudshell.png">
</kbd>

In the following, all the commands are typed in that shell.
We will make use of many `kubectl` commands. If interested, you can read
[an overview](https://kubernetes.io/docs/reference/kubectl/overview/)
of this command line tool for Kubernetes.

## Install argo as a workflow engine

While jobs can also be run manually, a workflow engine makes defining and
submitting jobs easier. In this tutorial, we use
[argo](https://argoproj.github.io/argo/quick-start/).
Install it into your working environment with the following commands
(all commands to be entered into the cloud shell):

While jobs can also be run manually, a workflow engine makes defining and submitting jobs easier. In this tutorial, we use [argo quick start](https://argoproj.github.io/argo-workflows/quick-start/) page to install it:

```
kubectl create clusterrolebinding YOURNAME-cluster-admin-binding --clusterrole=cluster-admin --user=YOUREMAIL@gmail.com
kubectl create ns argo
kubectl apply -n argo -f https://raw.githubusercontent.com/argoproj/argo-workflows/stable/manifests/quick-start-postgres.yaml
```

Download argo CLI:

```
# Download the binary
curl -sLO https://github.com/argoproj/argo-workflows/releases/download/v3.1.2/argo-linux-amd64.gz

# Unzip
gunzip argo-linux-amd64.gz

# Make binary executable
chmod +x argo-linux-amd64

# Move binary to path
sudo mv ./argo-linux-amd64 /usr/local/bin/argo

# Test installation
argo version
```

```bash
kubectl create ns argo
kubectl apply -n argo -f https://raw.githubusercontent.com/argoproj/argo/stable/manifests/quick-start-postgres.yaml
curl -sLO https://github.com/argoproj/argo/releases/download/v2.11.1/argo-linux-amd64.gz
gunzip argo-linux-amd64.gz
chmod +x argo-linux-amd64
sudo mv ./argo-linux-amd64 /usr/local/bin/argo
```

This will also install the argo binary, which makes managing the workflows
easier.

> ## Reconnecting after longer time away
>
> In case you leave your computer, you might have to reconnect to the CloudShell
> again, and also on a different computer. If the `argo` command is not found,
> run the command above again starting from the `curl` command.
>
{: .callout}

You need to execute the following command so that the argo workflow controller
has sufficient rights to manage the workflow pods.
Replace `XXX` with the number for the login credentials you received.

```bash
kubectl create clusterrolebinding cern-cms-cluster-admin-binding --clusterrole=cluster-admin --user=cms-gXXX@arkivum.com
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

## Yaml Files

Another way of creating components in a K8s cluster is trough yaml files.  These are intuitive, logical and configurable, although very picky about identation.

Let's take a look at one of these files, [nginx-deployment.yaml](https://gitlab.com/nanuchi/youtube-tutorial-series/-/raw/master/kubernetes-configuration-file-explained/nginx-deployment.yaml):

~~~
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.16
        ports:
        - containerPort: 8080
~~~
{: .language-yaml}

Let's delete our previous deployment and deploy using the yaml file:

* Delete deployments

```
kubectl delete deployment mynginx-depl
```

* Deploy using yaml files

```
kubectl apply -f nginx-deployment.yaml
```

## Namespaces

Namespaces are a kind of reservations in your K8s cluster.  Let's create one for the Argo workflow we will user

```
kubectl create ns <NAMESPACE>
```



## Storage volumes

If we run some application or workflow, we usually require a disk space where to dump our results.  There is no persistent disk by default, we have to create it.

You could create a disk clicking on the web interface above, but lets do it faster in the command line.

Create the volume (disk) we are going to use

```
gcloud compute disks create --size=100GB --zone=us-central1-c gce-nfs-disk-1
```

Set up an nfs server for this disk:

```
wget https://cms-opendata-workshop.github.io/workshop2021-lesson-cloud/files/001-nfs-server.yaml
kubectl apply -n argo -f 001-nfs-server.yaml
```

Set up a nfs service, so we can access the server:

```
wget https://cms-opendata-workshop.github.io/workshop2021-lesson-cloud/files/002-nfs-server-service.yaml
kubectl apply -n argo -f 002-nfs-server-service.yaml
```

Let's find out the IP of the nfs server:

```
kubectl get -n argo svc nfs-server |grep ClusterIP | awk '{ print $3; }'
```

Let's create a *persisten volume* out of this nfs disk.  Note that persisten volumes are not namespaced they are available to the whole cluster.

We need to write that IP number above into the appropriate place in this file:

```
wget https://cms-opendata-workshop.github.io/workshop2021-lesson-cloud/files/003-pv.yaml
```

~~~
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-1
spec:
  capacity:
    storage: 100Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: <Add IP here>
    path: "/"
~~~
{: .language-yaml}

Deploy:

```
kubectl apply -f 003-pv.yaml
```

Check:

```
kubectl get pv
```

Apps can claim persistent volumes through *persistent volume claims* (pvc).  Let's create a pvc:

```
wget https://cms-opendata-workshop.github.io/workshop2021-lesson-cloud/files/003-pvc.yaml
kubectl apply -n argo -f 003-pvc.yaml
```
Check:

```
kubectl get pvc -n argo
```

Now an argo workflow coul claim and access this volume with a configuration like:

~~~
# argo-wf-volume.ysml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: test-hostpath-
spec:
  entrypoint: test-hostpath
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: nfs-<NUMBER>
  templates:
  - name: test-hostpath
    script:
      image: alpine:latest
      command: [sh]
      source: |
        echo "This is the ouput" > /mnt/vol/test.txt
        echo ls -l /mnt/vol: `ls -l /mnt/vol`
      volumeMounts:
      - name: task-pv-storage
        mountPath: /mnt/vol
~~~
{: .language-yaml}


Now the workflow could access the disk. However is rather cumbersome to get the ouput outside the cluster.  One of he best ways to do that is to follow similar procedures to set up an http web server for example.

As you can see all these tasks take time.  The good news is that there are tools to make the cluster creation and the setting up of volumes, pvs, pvcs, servers, services, etc., automatic.  This is what we will be doing for you (using `Terraform`) for the upcoming hands-on part of this lesson.

You will continue learning about this, hands-on, tomorrow, but concentrating on analysis workflows.



## Run a simple test workflow

To test the setup, run a simple test workflow with

```bash
argo submit -n argo --watch https://raw.githubusercontent.com/argoproj/argo/master/examples/hello-world.yaml
argo list -n argo
argo get -n argo @latest
argo logs -n argo @latest
argo delete -n argo @latest
```

Please mind that it is important to delete your workflows once they have
completed. If you do not do this, the pods associated with the workflow
will remain scheduled in the cluster, which might lead to additional charges.
You will learn how to automatically remove them later.

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


{% include links.md %}
