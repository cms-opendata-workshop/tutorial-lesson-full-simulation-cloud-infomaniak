+++
title = "Production Run"
weight = 40
teaching = 15
exercises = 10
questions = ["How to do the processing with the larger amount of events?"]
objectives = ["Run the production workflow and create the final dataset"]
keypoints = ["Learn to produce a production scale dataset"]
+++

## Order a production size cluster

After the last chapter, you should have an understanding of how many nodes are needed in a cluster to process a dataset for your specific needs.

{{<tabs>}}
{{<tab name="Terraform" selected="true">}}

First, delete the test cluster:

```bash
terraform destroy
```

Then edit the `main.tf` to order a cluster with the correct amount of resources. You should have planned the size of your resources in the last chapter.

```terraform
resource "infomaniak_kaas_instance_pool" "workers" {
    public_cloud_id             = infomaniak_kaas.cluster.public_cloud_id
    public_cloud_project_id     = infomaniak_kaas.cluster.public_cloud_project_id
    kaas_id                     = infomaniak_kaas.cluster.id
    name                        = "worker-pool"
    flavor_name                 = "a4-ram16-disk80-perf1"
    availability_zone           = "dc3-a-04"
    min_instances               = 2 # EDIT THIS ACCORDINGLY
    max_instances               = 2 # EDIT THIS ACCORDINGLY
}
```

Order the cluster:

```bash
terraform plan
terraform apply
```

{{</tab>}}


{{<tab name="Web Interface">}}

First, delete the test cluster. Open the [Manager Page](https://manager.infomaniak.com) and select your Public Cloud and the project. Go to Kubernetes and delete your cluster.

After that, create a new one just like you created the test cluster, but with the correct amount of nodes. This you should have calculated in the last chapter.

{{</tab>}}
{{</tabs>}}


## Prepare the cluster

Run the same preparation steps as with the test cluster

```bash
export KUBECONFIG=/path/to/your/kubeconfig
```

```bash
kubectl create ns argo

kubectl apply -n argo --server-side -f https://github.com/argoproj/argo-workflows/releases/download/v4.0.1/install.yaml
kubectl apply -f manifests/

kubectl create secret generic s3-credentials \
  --from-literal=S3_ACCESS_KEY_ID='<the value of the access field>' \
  --from-literal=S3_SECRET_ACCESS_KEY='<the value of the secret field>' \
  -n argo
```

{{<callout type="note" title="Metrics Server">}}
If you want to use the Metrics Server on your production workflow, you have to configure it again for the new cluster:
```bash
git clone https://github.com/cms-opendata-processing-tasks/WorkflowUtils.git
cd WorkflowUtils

kubectl apply -f components.yaml
```
{{</callout>}}

## Run the workflow

Connect to the OpenStack by running
```bash
source PCP-XXXXXXX-openrc.txt
```

{{<tabs>}}
{{<tab name="GEN" selected="true">}}
Check that your data fragment is in place:
```bash
openstack object list your_storage
```

If not, upload it from your working directory:
```bash
openstack object create your_storage XXX-RunXXXXXX-YYYYY-fragment.py --name FullSim/my-dataset/XXX-RunXXXXXX-YYYYY-fragment.py
```

Remember to fix the fragment file name to the `FullSimulationArgoWorkflow/cms-simulation-process/run-pp-simulation.yaml` parameters!
{{</tab>}}
{{<tab name="LHE GEN">}}

Check that your LHE GEN step's root files are in place:
```bash
openstack object list your_storage
```

If not, upload your LHE GEN step root files to the OpenStack Object Storage:
```bash
swift upload your_storage GEN/ --object-name FullSim/my-dataset/GEN/
```
{{</tab>}}
{{</tabs>}}

## Monitor the workflow run

To monitor the status of the workflow, you can run:

```bash
argo get @latest -n argo
```

If you want to further investigate a certain pod, you can run:

```bash
kubectl logs -n argo simulation-process-xxxxx-step-name-xxxxxxxxxx

kubectl describe -n argo pod/simulation-process-xxxxx-step-name-xxxxxxxxxx
```


<!--TODO: run the jobIndexes independently > then you can run a certain failed index afterwards > instructions for that -->