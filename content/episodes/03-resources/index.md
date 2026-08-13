+++
title = "Computing Resources"
weight = 30
teaching = 10
exercises = 10
questions = ["What kind of cluster do I need to process a full dataset?", "How long will the processing take with these resources?", "How much will the resources cost?"]
objectives = ["Determine how many nodes are needed for your project and how much they cost.", "Understand all the different devices, such as storage and nodes, that contribute to the total costs.", "Get a grasp of the price range of data processing with the workflow."]
keypoints = ["The resources needed for your workflow depend on the number of events and how fast you want your results.", "In most cases, you can process a reasonable sized dataset with the free credits."]
+++

{{< callout type="prereq" title="Prerequisites" >}}
To get to know the basics of cloud computing resources, please complete the [Setup]({{< relref "/learners/setup" >}}) and read more on [the Kubernetes website](https://kubernetes.io/docs/concepts/architecture/).

By now, you should have decided how many events there will be in your final dataset. This number is used as a starting point to determine the resources needed for the final workflow.
{{< /callout >}}

## Nodes

The number of nodes needed depends on the number of jobs you want to divide your process in. The number of jobs, in turn, depends on the total event number of the dataset and the rate at which you want the dataset to finish. More jobs means it will finish faster but that will require more nodes.

One processing job should be a maximum of 1600 events. This is because the DIGI2RAW step adds the pileup from a dataset, whose files contain 1600 events. The workflow only assigns one pileup file per job, and that file has 1600 events. If the job had more events, not all interactions would be assigned pileup and the result would not be valid. There is no lower limit to the amount of events the job can have.

A single job uses a lot of memory, so on a node with 16 GB RAM, there can only be two jobs at a time. The job also requires ~1 vCPU. Unfortunately, Infomaniak does not currently offer 2 vCPU 16 GB RAM nodes, so we will be using the 4 vCPU 16 GB RAM nodes.

Next, calculate the minimum amount of nodes you will need for your final workflow using the following formulas:

$$
\begin{align*}
\text{number of jobs} = \frac{\text{events in total}}{1600} \\
\text{number of nodes} = \frac{\text{number of jobs}}{2}
\end{align*}
$$

If you want the workflow to finish faster, you can assign less events per job and order more nodes, as long as one node only has max. 2 jobs at a time.

{{<callout type="note" title="Expand the limits of the Public Cloud">}}
As default, the Infomaniak Public Cloud has limits that do not allow you to make a cluster with more than 10 nodes with a maximum of 64 GB RAM between all of them.

In order to create a larger cluster to process your data, you should contact Infomaniak about expanding the limits of your Public Cloud project. Before contacting, study this chapter until the end, to have a clear plan of what size limits you will need to process your data.

The larger clusters can still be used to process a reasonably sized dataset within the free token range, as we will calculate later.
{{</callout>}}

## Volumes

Volumes enable the steps of the workflow to have common files between different containers. Since every step is always a new container, the common volume enables the containers to relay the files onto the next step and the next container.

The volumes are mounted in the container as external devices and act as terminals for the intermediate root files.

Rough file size estimation per step, if the files contain 1600 events:

| Step     | Size / 1 600 events | 
| -------- | ------------------- |
| GEN      | 200 MB              |
| SIM      | 1 GB                |
| DIGI2RAW | 3 GB                |
| HLT      | 2 GB                |
| RECO     | 500 MB              |
| PAT      | 100 MB              |
| NANO     | 50 MB               |

After each step is completed, the files that are not needed get deleted. For example, once the DIGI2Raw step is completed, the GEN steps root files can be deleted and so on.

This would mean that at most the volume would have roughly 5 gigabytes stored on it during a single job. Each node has their own volume, but a node has two jobs, so this amount has to be at least doubled. The sizes are only rough estimates, which is why the volumes created in the last chapter were 15 gigabytes in size.

Having these volumes in your OpenStack area, even while empty, is not free. This is why it's important to delete the volumes once they are not in use.

Currently, we order one volume per processing node, because the option `ReadWriteMany` for NFS (Network File System), which would enable the nodes to write all on the same volume, is not available on Infomaniak platforms.


## What does the cluster cost in total?

In addition to nodes and volumes, you need a control plane for the cluster. These three components combined have prices shown below:

| Product                                                | Price (CHF)                | Amount needed       |
| ------------------------------------------------------ | -------------------------- | ------------------- |
| **Control plane:** Cluster Dedicated 4                 | 0.04 CHF / hour            | 1                   |
| **Nodes:** 4 vCPU, 16 GB RAM, 80 GB Disk Space         | 0.03191 CHF / hour         | 1 for every 2 jobs  |
| **Volumes**: performance level 1, i.e. writes 200 MB/s | 0.00012 CHF / GB / hour    | ~ 7 GB for every job|

For instance, if you are processing a dataset with 50 000 events, the total cost would then be about 0.60 CHF / hour.

{{<callout type="note" title="Prices might be obsolete">}}
Keep in mind that the prices might change. Visit [Infomaniak's official website](https://www.infomaniak.com/en/hosting/public-cloud/prices) for the current information.
{{</callout>}}

## How long are the nodes in use?

The time of the processing depends on how many events are in one job. If a user wants to get the dataset more quickly, they should order a cluster with more nodes. That way the jobs will take less time because they are more spread out.

For example with 1600 events per job the steps take about:

| Step            | Time estimation         |
| --------------- | ----------------------- |
| GEN             | 1 h                     |
| SIM             | 4 h                     |
| DIGIPremix      | 0,5 h                   |
| HLT             | 1 h                     |
| RECO            | 1 h                     |
| PAT             | 10 min                  |
| NANO            | 5 min                   |
| Total:          | ~ 8 h                   |

These times may vary on multiple reasons

- If you are running the workflow on newly created nodes, they don't yet have the container images needed and the run will take 5-15 minutes longer because of the installation of the images
- If a job fails at a certain step, depending on how long the step takes, it could add another 1-4 hours to the processing time


Because the uncertain runtime, it is most common to leave the workflow running for the night, which would mean the cluster is charged for approximately 16 hours.


## So the total price would be...

### Example 1

Objective: Dataset with 50 000 events with 800 jobs per event, i.e. half of the process time

- 50 000 events needs at least 32 nodes.
- This results in most of the jobs having 781 events per job except two having 789, i.e. roughly 800 events per job
- The volumes would be half the size of 1600 event jobs. Therefore the cost for volumes is the same as for 1600 events/job.
- The full workflow takes approximately 4 hours if no fails, if only one step fails 8 hours

Let's say the user leaves the workflow running during the night, the cluster is up for 16 hours:

**Total price: 18 CHF**


### Example 2

Objective: Dataset with 200 000 events with 1600 events per job

- Now 200 000 events would mean 63 nodes
- Roughly 1600 events per job
- The full workflow would take 8 hours, 12 if retries once

Therefore let's calculate with the same 16 hour approximate:

**Total price: 35 CHF**

Keep in mind that these are estimates. Your workflow could be slower or faster, retry several times or run succesfully already during a working day. Or you might have problems which require you to order a smaller cluster for sometime for troubleshooting.

This chapter is only to give you an estimate of the costs. Although usually, the processing is possible using the Infomaniak free credits, since the costs for one workflow is significantly under 100 CHF.