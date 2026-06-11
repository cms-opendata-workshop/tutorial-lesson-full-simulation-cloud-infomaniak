+++
title = "Introduction"
weight = 10
teaching = 10
exercises = 10
questions = ["What does this workflow do?", "How to get started on this workflow?"]
objectives = ["Understand what the workflow does and who is it for."]
keypoints = ["This workflow starts from fragments and processes data using the CMS software and produces a simulated dataset in either NANOAOD format or for HeavyIons the RECO format."]
+++

# About the workflow

The goal of this workflow is to use public cloud resources, here Infomaniak resources, to process data following certain steps, and ultimately produce a simulated dataset ready for data analysis.

# Why public cloud?

Data processing can, in theory, be done with

- Institute's resources
- Local resources i.e. your own computer
- Public resources

This tutorial shows how to create a simulated dataset without access to the two former and only using the latter. Infomaniak is an affordable Switzerland based cloud provider, whose resources we will be using throughout this tutorial.

# Getting started

### Prerequisites

To get an Infomaniak Kubernetes cluster and Argo setup on it, follow [this tutorial](https://cms-opendata-workshop.github.io/tutorial-lesson-cloud-processing-infomaniak/)

### 1. Kubernetes and Argo

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

### 2. Clone the workflow

Clone the workflow repository on your computer using:

```bash
git clone git@github.com:cms-opendata-processing-tasks/FullSimulationArgoWorkflow.git
```

- clone the repo

- get fragments

## 1. Run setup checks first

Before editing content, confirm your toolchain from the
[Setup]({{< relref "/learners/setup" >}}) page.

Then run:

```bash
hugo version
hugo server
```

This works without Go because the template commits a vendored module tree in `_vendor/`.

If setup is unclear, use the shared docs:

- [hugo-styles Quickstart](https://oer-particle-physics.github.io/hugo-styles/docs/quickstart/)
- [hugo-styles Troubleshooting](https://oer-particle-physics.github.io/hugo-styles/docs/troubleshooting/)

## 2. Update `hugo.toml` early

Edit these fields before replacing episode content:

| Field                                       | Why it matters                                                                                            |
| ------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| `baseURL`                                   | Production URL for deployed pages and canonical links, for example `https://<account>.github.io/<repo>/`. |
| top-level `title`                           | Site title used in browser/title surfaces.                                                                |
| `[params.lesson].title`                     | Lesson title shown in theme components.                                                                   |
| `[params.lesson].tagline` and `description` | Homepage framing and metadata summary.                                                                    |
| `[params.lesson].contact`                   | Contact shown in lesson metadata contexts.                                                                |
| `[params.lesson].repo`                      | Footer source link target in the UI.                                                                      |
| `[params.lesson].editBranch`                | Branch used for “Edit this page” links.                                                                   |
| `[params.versioning]`                       | Controls whether the deployment workflow publishes only `Latest` or also archived branch/tag builds.      |
| `[[menus.main]]` GitHub `url`               | Top-nav GitHub link target.                                                                               |

If you plan to deploy on GitHub Pages, enable Pages in the repository settings and choose
`GitHub Actions` as the source before the first push to `main`.
The included workflow deploys on pushes to `main`, so that first push should already have a configured Pages target.

{{< callout type="warning" title="Keep these as-is unless you know why" >}}
Do not remove the `module.imports` block that points to `github.com/oer-particle-physics/hugo-styles`.
That import is what provides the shared layouts, shortcodes, and supporting behavior.
{{< /callout >}}

## 3. Replace template content with your lesson

After metadata is set:

- replace this sample episode with your first real episode
- set episode order with front matter `weight` values (`10`, `20`, `30`, ...) rather than filename prefixes
- update `content/_index.md` homepage copy
- add or rename glossary/profile pages if needed

For structure and conventions, use:

- [Authoring Guide](https://oer-particle-physics.github.io/hugo-styles/docs/authoring/)
- [Components](https://oer-particle-physics.github.io/hugo-styles/docs/components/)
- [Front Matter](https://oer-particle-physics.github.io/hugo-styles/docs/frontmatter/)
- [Hextra Features for Physics Lessons](https://oer-particle-physics.github.io/hugo-styles/docs/hextra-features/)
- [Deployment](https://oer-particle-physics.github.io/hugo-styles/docs/deployment/)

{{< challenge title="Local vs shared" >}}
List two things that should stay in this lesson repository and two things that should stay in the shared module.

{{< hint >}}
Think about what is specific to your lesson topic versus what should stay reusable across many lessons.
{{< /hint >}}

{{< solution >}}
Lesson prose, schedule, glossary/profile content, and local branding should stay in this repository.
Shared layouts, pedagogy shortcodes, CSS/JS behavior, and validation tooling should stay upstream in `hugo-styles`.
{{< /solution >}}
{{< /challenge >}}

{{< instructor >}}
Point maintainers to the update guide early so they know updates should arrive through module version bumps:
[Updating Downstream Lessons](https://oer-particle-physics.github.io/hugo-styles/docs/updates/).
{{< /instructor >}}
