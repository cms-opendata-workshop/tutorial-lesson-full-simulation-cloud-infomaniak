+++
title = "Production Run"
weight = 40
teaching = 15
exercises = 10
questions = ["How to do the processing with the larger amount of events?"]
objectives = ["Run the production workflow and create the final dataset"]
keypoints = [" "]
+++


Based on how many events you are going to simulate, you should have ordered a suitable cluster in the last chapter. 

Connect again to the cluster by moving the Kubeconfig to your working directory and export the path to Kubeconfig:
```bash
mv /path/to/pck-xxxxxxx-kubeconfig /path/to/workingdir
export KUBECONFIG=/path/to/workingdir/pck-xxxxxxx-kubeconfig
```

Create the argo namespace:
```bash
kubectl create ns argo
```

Attach your s3credentials to this new cluster:
```bash
kubectl create secret generic s3-credentials \
  --from-literal=S3_ACCESS_KEY_ID='<the value of the access field>' \
  --from-literal=S3_SECRET_ACCESS_KEY='<the value of the secret field>' \
  -n argo
```

Connect to the OpenStack by running
```bash

```