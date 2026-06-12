+++
title = "Getting Started"
weight = 20
teaching = 15
exercises = 10
questions = ["How to run the workflow?"]
objectives = ["Run a succesful workflow with a small number of events.", "Understand what is needed from the user in the workflow."]
keypoints = ["Before the workflow you have to clone the code, create a cluster and have a data fragment.", "The workflow is started with `argo submit`"]
+++

## 1. Clone the workflow

Clone the workflow repository on your computer using:

```bash
git clone git@github.com:cms-opendata-processing-tasks/FullSimulationArgoWorkflow.git
```

## 2. Kubernetes and Argo

Once you have created a cluster on the Infomaniak Dashboard and it is up and running, you can download the Kubeconfig file to your computer. Move the config to your working directory and set it to your environment variables:

```bash
export KUBECONFIG=/path/to/your/pck-xxx-kubeconfig
```

Check the connection to the cluster

```bash
kubectl cluster-info

Kubernetes control plane is running at https://83.xxxx
CoreDNS is running at https://83.xx:xxxx/api/v1/namespaces/kube-system/services/pck-yxulp7c-addon-coredns:udp-53/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

Next apply the argo tools:

```bash
kubectl create namespace argo
kubectl apply -n argo --server-side -f https://github.com/argoproj/argo-workflows/releases/download/v4.0.1/install.yaml
kubectl apply -f manifests/
```

In the [Setup tutorial](https://cms-opendata-workshop.github.io/tutorial-lesson-cloud-processing-infomaniak/), you created s3credentials. Give them now to the cluster:

```bash
kubectl create secret generic s3-credentials \
  --from-literal=S3_ACCESS_KEY_ID='<the value of the access field>' \
  --from-literal=S3_SECRET_ACCESS_KEY='<the value of the secret field>' \
  -n argo
```

## 3. Fragments <!--Is this a universal step? Should this be in the different sim case chapters?-->

To generate the dataset, the workflow starts with one Python fragment file. Depending on if you want to create your own dataset or, for example, duplicate some set on the [Open Data Portal](https://opendata.cern.ch) you either write your own fragment or get an existing one from the internet.

<!--Check if this is true and add instructions how to create fragments-->

If you want to get a fragments file from the Open Data Portal, choose your dataset and go to the first step's production script. There should be a `curl` command, e.g.

<!--Screenshots how to find fragment curl command-->

```bash
curl -s -k https://cms-pdmv-prod.web.cern.ch/mcm/public/restapi/requests/get_fragment/XXX-RunXXXXXX-YYYYY --retry 3 --create-dirs -o Configuration/GenProduction/python/XXX-RunXXXXXX-YYYYY-fragment.py
[ -s Configuration/GenProduction/python/XXX-RunXXXXXX-YYYYY-fragment.py ]
```

Once the fragments file is in your working file, copy it to the cloud's object storage. You should be familiar with Infomaniak object storage after the [Setup Tutorial](https://cms-opendata-workshop.github.io/tutorial-lesson-cloud-processing-infomaniak/)

```bash
openstack object create your_storage XXX-RunXXXXXX-YYYYY-fragment.py --name FullSim/parallel-testing/XXX-RunXXXXXX-YYYYY-fragment.py
```
