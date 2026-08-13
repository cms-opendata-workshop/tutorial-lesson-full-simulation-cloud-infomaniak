+++
title = "Production Run"
weight = 40
teaching = 20
exercises = 35
questions = ["How to do the processing with the larger amount of events?"]
objectives = ["Order a production size cluster from the Infomaniak Public Cloud", "Run the production workflow and create the final dataset"]
keypoints = ["To run a large number of events, the cluster needs a lot of disk space in the form of volumes.", "In most cases the processing takes roughly one working day, after which the dataset can be downloaded from OpenStack Object Storage."]
+++

{{<callout type="prereq" title="Infomaniak resource limits need to be expanded">}}
By default the free tier of Infomaniak resources are not sufficient to process more than 8 jobs of 1600 events. This is because the Public Cloud has default limits that allow only 64 GB of RAM and one node needs 16. I.e. you could create a cluster of max. 4 nodes in this free tier, even if the free tokens would cover more.

If you want to process a bigger dataset, you should contact Infomaniak if they could expand the limits on your area.
{{</callout>}}

## Order a production size cluster

After the last chapter, you should have an understanding of how many nodes are needed in a cluster to process a dataset for your specific needs.

Choose your method of ordering a cluster, Terraform or the Manager dashboard:

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

Add more volumes based on the numbers of nodes you are ordering:

```terraform
# Copy this resource block and rename the node numbers until you have the same amount of volumes as nodes

resource "openstack_blockstorage_volume_v3" "volume16" {
  name              = "volume_16"
  description       = "Volume for computing node 16"
  size              = 10
  availability_zone = "nova"
}
```

Order the cluster:

```bash
terraform plan
terraform apply
```

After the `terraform apply` has run, remember to extract the output for the new cluster:

```bash
terraform output -raw kubeconfig > ./kubeconfig
```

{{</tab>}}


{{<tab name="Web Interface">}}

First, delete the test cluster:

Open the [Manager Page](https://manager.infomaniak.com) and navigate to Cloud Computing > Kubernetes. Select your test cluster, click on "Manage" and then "Delete cluster".

After that, create a new one just like you created the test cluster in [Setup]({{< relref "/learners/setup" >}}), but with the amount of nodes you have calculated in the last chapter.

You should have the object storage container remaining from the second chapter, but more volumes are needed in a larger workflow.

```bash
openstack volume create volume_3 --description "Volume for computing node 3" --size 15 volume_3
openstack volume create volume_4 --description "Volume for computing node 4" --size 15 volume_4
# and so on
```

{{</tab>}}
{{</tabs>}}


## Prepare the cluster

Run the same preparation steps as with the test cluster.

Connect to the cluster:

```bash
export KUBECONFIG=/path/to/your/kubeconfig
```

Setup argo:

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
cd WorkflowUtils/memory_scan

kubectl apply -f components.yaml
```
{{</callout>}}

## Mount the volumes

Sign in to the OpenStack area if not already:
```bash
source PCP-XXXXXXX-openrc.sh
```

Edit the `persistent_volume.yaml` and `persistent_volume_claim.yaml` files to correspond to the new amount of volumes.

Add new volumes and their UUIDs in the `persistent_volume.yaml` file:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-3
spec:
  capacity:
    storage: 15Gi
  accessModes:
    - ReadWriteMany
  csi:
    driver: cinder.csi.openstack.org
    ## EDIT BELOW: replace the placeholder 'xxxxxxxx-xxxx-xxxx' with your volumes UUID
    volumeHandle: xxxxxxxx-xxxx-xxxx
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  claimRef:
    name: pvc-3
    namespace: argo
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-4
spec:
  capacity:
    storage: 15Gi
  accessModes:
    - ReadWriteMany
  csi:
    driver: cinder.csi.openstack.org
    ## EDIT BELOW: replace the placeholder 'xxxxxxxx-xxxx-xxxx' with your volumes UUID
    volumeHandle: xxxxxxxx-xxxx-xxxx
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  claimRef:
    name: pvc-4
    namespace: argo
---
# and so on
```

Again, you can find the UUIDs of the newly created volumes by running:

```bash
openstack volume list
```

Increase the amount of Persistent Volume Claims in the `persistent_volume_claim.yaml` file:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-3
  namespace: argo
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  storageClassName: manual
  volumeName: pv-3
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-4
  namespace: argo
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  storageClassName: manual
  volumeName: pv-4
---
# and so on
```

Apply the new volume configurations:

```bash
kubectl apply -f persistent_volume.yaml
kubectl apply -f persistent_volume_claim.yaml
```

## Edit the parameters

With the larger cluster, some parameters need to be edited compared to the run on the test cluster.

Update the value of `nNodes` and `totEvents` according to your situation. If you want the test run and the production run to save their files in different folders, edit also the `dataName`. Remember to use this new data name when uploading input files in the next step.

Add the mounts of the additional volumes. The volumes are mentioned in two places in the `run-pp-simulation.yaml` file.

```yaml
spec:
  entrypoint: cms-full-sim
  serviceAccountName: argo-service-account
  volumes:
    - name: volume1
      persistentVolumeClaim:
        claimName: pvc-1
    - name: volume2
      persistentVolumeClaim:
        claimName: pvc-2
    - name: volume3
      persistentVolumeClaim:
        claimName: pvc-3
    # and so on
```

Add a mention of your additional volumes to the final `merge-result-files` step as well:

```yaml
- name: merge-result-files-template
  ...
  script:
          image: gitlab-registry.cern.ch/cms-cloud/root-vnc:latest
          command: [bash]
          volumeMounts:
            - name: volume1
              mountPath: /data/1
            - name: volume2
              mountPath: /data/2
            - name: volume3
              mountPath: /data/3
            # and so on
```

Get the new nodeName object by running:

```bash
kubectl get nodes -o jsonpath='{range .items[*]}{"{\"nodeName\":\""}{.metadata.name}{"\"}"}{","}{end}' | sed 's/,$//' | sed "s/\"/'/g"
```

Copy the output json to the workflow:
```yaml
### EDIT BELOW: Set here the list of your node names, i.e. the output of kubectl get nodes -o jsonpath='{range .items[*]}{"{\"nodeName\":\""}{.metadata.name}{"\"}"}{","}{end}' | sed 's/,$//' | sed "s/\"/'/g"
      - name: nodeNames
        value: [{'nodeName':'pck-xxxxxxx'},{'nodeName':'pck-xxxxxxx'}]
```

## Upload input files

{{<tabs>}}
{{<tab name="GEN" selected="true">}}
Check that your data fragment is in place:
```bash
openstack object list mystorage
```

If not, upload it from your working directory:
```bash
openstack object create mystorage <name-of-your-fragment>.py --name FullSim/my-dataset/input/<name-of-your-fragment>.py
```

Remember to have the correct `inputFileName` `run-pp-simulation.yaml` file's parameters!
{{</tab>}}
{{<tab name="LHE GEN">}}

Check that your data fragment and gridpack are in place:
```bash
openstack object list mystorage
```

If not, upload them from your working directory:
```bash
openstack object create mystorage <name-of-your-fragment>.py --name FullSim/my-dataset/input/<name-of-your-fragment>.py

openstack object create mystorage <gridpack-name>_slc7_amd64_gcc700_CMSSW_10_6_0_tarball.tar.xz --name FullSim/my-dataset/input/<gridpack-name>_slc7_amd64_gcc700_CMSSW_10_6_0_tarball.tar.xz
```

Remember to fix the fragment file name and the gridpack file name to the `FullSimulationArgoWorkflow/cms-simulation-process/run-pp-simulation.yaml` parameters!

Also check that the path to the gridpack in your fragment file is `/code/CMSSW_10_6_30/src/<gridpack-name>_slc7_amd64_gcc700_CMSSW_10_6_0_tarball.tar.xz`
{{</tab>}}
{{</tabs>}}


## Monitor the workflow run

Start the processing:

```bash
argo submit -n argo cms-simulation-process/run-pp-simulation.yaml
```

To monitor the status of the workflow, you can run:

```bash
argo get @latest -n argo
```

If you want to further investigate a certain pod, you can ge the pod name from the output of the last command and run for that pod either:

```bash
kubectl logs -n argo <pod-name>
# and/or
kubectl describe -n argo <pod-name>
```

## Download the output files

Once the workflow has run, it saves the output files in to OpenStack Object Storage. There are three types of files as output: MiniAOD, NanoAOD and the large result files. The MiniAOD and NanoAOD are saved from each parallel job into their own folders in the Object Storage container. Finally the large result files are a product of the `merge-result-files` step, which simply combines the small NanoAOD files into fewer, maximum 2 GB files, which are easier to move around.

The storage and its files can be accessed from either the command line or from the OpenStack [web interface](https://api.pub2.infomaniak.cloud), where you sign in with the PCU-XXXXX username from the OpenStack Access.

{{<tabs>}}
{{<tab name="Terminal" selected="true">}}

Inspect what files are saved in your container:
```bash
swift list mystorage
```

Install a file on your device:
```bash
swift download mystorage <path-to-file> --output <output-name>
```

If you run
```bash
openstack container list
```
you might notice a container of a name `mystorage+segments`. This is a behaviour of swift and OpenStack, when it creates segments to make the writing of large files more efficiently. This happens especially in the PAT step of the workflow, when the MiniAOD files are saved in the object storage. Read more about how large files are uploaded in to the object storage [here](https://docs.openstack.org/swift/pike/overview_large_objects.html). After the workflow has finished, and you have downloaded the results of the segments are useless, so you can delete them by:

```bash
openstack container delete mystorage+segments
```

It is recommended to delete this container, since it has a passive cost as do all other cloud resources.



{{</tab>}}
{{<tab name="Web Interface">}}

Go to the [OpenStack Dashboard](https://api.pub2.infomaniak.cloud) and navigate to Object Store > Containers in the side bar.

This opens a display of all your object storages, click on the one you have been using for the processing workflow and navigate to your dataset. You will find the output files in folders such as MiniAOD, NanoAOD and results.

{{</tab>}}
{{</tabs>}}


## Clean up

Once you have installed your results from the Object Storage and don't need the cluster anymore, delete all resources. Otherwise the resources will keep billing your account.

First, delete the workflow:
```bash
argo delete @latest -n argo
```

Delete all the persistent volumes and persistent volume claims:

```bash
kubectl delete pvc -n argo pvc-1 pvc-2 pvc-3 # and so on
```

```bash
kubectl delete pv pv-1 pv-2 pv-3 pv-4 pv-5 # and so on
```
<!--To-do: Are there better commands on deleting all pvs and pvcs -->


You can check that the persistent volumes don't exist anymore by running all of these:
```bash
kubectl get pv
kubectl get pvc -n argo
```


When these commands return "No resources found" the volumes and cluster are ready for deleting.

If you use Terraform with admin privileges:
```bash
terraform destroy
```

Or you can delete them manually by running
```bash
openstack volume delete volume_1
openstack volume delete volume_2
```
and after that deleting the cluster from the [web interface](https://manager.infomaniak.com/).


