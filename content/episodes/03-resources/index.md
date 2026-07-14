+++
title = "Computing Resources"
weight = 30
teaching = 15
exercises = 10
questions = ["What kind of resources should my cluster have?", "How much will the resources cost?"]
objectives = ["Understand how many and what kind of nodes are needed for a certain project and how much they cost."]
keypoints = ["The resources needed for your workflow depend on the number of events and how fast you want your results."]
+++

{{< callout type="prereq" title="Prerequisites" >}}
To get to know the basics of cloud computing resources, please complete the [Setup]({{< relref "/learners/setup" >}}) and read more on [the Kubernetes website](https://kubernetes.io/docs/concepts/architecture/).
{{< /callout >}}

## This chapter is still a work in progress

Until the problems with disk space and memory allocation have been solved, the resources needed can't be determined. Once these challenges are solved, this chapter will be updated.

## About the nodes

One processing job should be a maximum of 800 events. The job takes at most 1 vCPU which means, that every node of 4 vCPUs can run 4 jobs simultaneously.

$$
\begin{align*}
\text{number of jobs} = \frac{\text{events in total}}{800} \\
\text{number of nodes} = \frac{\text{number of jobs}}{4}
\end{align*}
$$

Infomaniak offers different kinds of nodes, but currently, the 4 vCPU, 16 GB RAM and 80 GB disk ones are the best for this purpose. Mostly because 4 vCPU is the most one can get with a free tier, and the CMSSW container images are quite large and take a lot of disk space on the node.

To-do:
- Count from the memory plots how much is the least of memory it needs


## What do the clusters cost?

To run the workflow you need a cluster with a control plane and processing nodes that do the actual jobs. These are priced based on the time they are online.

| Product                                         | Price (CHF)         | Amount needed       |
| ----------------------------------------------- | ------------------- | ------------------- |
| **Control plane:** Cluster Dedicated 4          | 0.04 CHF / hour     | 1                   |
| **Nodes:** 4 vCPU, 16 GB RAM, 80 GB Disk Space  | 0.03191 CHF / hour  | 1 for every 4 jobs  |


For example if you are processing a dataset with 50 000 events, the total cost would be about 0.52 CHF / hour.

## How long are the nodes in use?

The time of the processing depends on how many events are in one job. If a user wants to get the dataset more quickly, they should order a cluster with more nodes. That way the jobs will take less time because they are more spread out.

For example with 800 events per job the steps take about

| **Step**        | **Time estimation**     |
| --------------- | ----------------------- |
| GEN             |                         |
| SIM             |                         |
| DIGIPremix      |                         |
| HLT             |                         |
| RECO            |                         |
| PAT             |                         |
| NANO            |                         |


File size estimation (These numbers are rounded up a lot)

| Step     | Size / 1 600 events | Total (32 jobia x 1600 = ~50k events) |
| -------- | -------- | ---------------------------- |
| GEN      | 200 MB   | 6 GB                         |
| SIM      | 1 GB     | 40 GB                        |
| DIGI2RAW | 3 GB     | 80 GB                        |
| HLT      | 2 GB     | 70 GB                        |
| RECO     | 500 MB   | 15 GB                        |
| PAT      | 100 MB   | 3 GB                         |
| NANO     | 50 MB    | 2 GB                         |


## So the total price would be...

### Example 1

Objective: Dataset with 50 000 events with least amount of resource costs


### Example 2

Objective: Dataset with 200 000 events with 800 events per job i.e. twice as fast.

