+++
title = "Setup"
weight = 10
+++


<!--
To-do: No dependence to Kati's tutorial with the setup steps and instead copy them here.
 -->

To get setup for this project follow two steps in another tutorial:

### 1. Install Infomaniak, OpenStack and Argo
Follow the [Setup](https://cms-opendata-workshop.github.io/tutorial-lesson-cloud-processing-infomaniak/learners/setup/) in CMS Open Data on Infomaniak -tutorial.

### 2. Order a cluster for testing

Follow the [Kubernetes Cluster](https://cms-opendata-workshop.github.io/tutorial-lesson-cloud-processing-infomaniak/episodes/04-cluster/) in CMS Open Data on Infomaniak -tutorial.

{{< callout type="note" title="Bonus Excercise" >}}
In the same tutorial the step [Set up a workflow](https://cms-opendata-workshop.github.io/tutorial-lesson-cloud-processing-infomaniak/episodes/05-workflow/) is very useful, if running Argo workflows is new to you. It is recommended to make sure that the `simple-test-s3.yaml` workflow in that tutorial runs on your cluster before continuing.
{{< /callout >}}

### 3. Enable Argo for the cluster

Once you have created a cluster on the Infomaniak Dashboard and it is up and running, you can download the Kubeconfig file to your computer. Move the config to your working directory and set it to your environment variables:

```bash
export KUBECONFIG=/path/to/your/pck-xxxxxxx-kubeconfig
```

```bash
kubectl create ns argo
```

### 4. Credentials for Argo to be able to use cloud storage

For Argo to be able to have access to your OpenStack storage in the Infomaniak cloud, one has to create credentials:

```bash
openstack ec2 credentials create
```

Save the output in `access` and `secret` fields. You will need them for the next command, which sets the credentials to the cluster.

```bash
kubectl create secret generic s3-credentials \
  --from-literal=S3_ACCESS_KEY_ID='<the value of the access field>' \
  --from-literal=S3_SECRET_ACCESS_KEY='<the value of the secret field>' \
  -n argo
```


### Next step

Once you have a working environment, continue to
[Introduction]({{< relref "/episodes/01-introduction" >}})
