# **Helm** - Kubernetes Package Manager

Deploying Kubernetes application which involve creating multiple resources – such as pods, services, deployments, and replicasets which requires YAML manifest file. But this can be very complex.

[Helm](https://helm.sh) is the package manager for Kubernetes. Like any other package managers(yum, apt), Helm provide a standard way to deploy, configure, upgrade, and run an Kubernetes application in a matter of minutes.

You can check [this](https://github.com/C123R/kubernetes-hands-on/tree/master/helm-example/nginx) sample helm chart which can deploy nginx along with service and configmap.

## Installation

```sh
brew install kubernetes-helm
```

Others can follow instructions from [here](https://helm.sh/docs/using_helm/#install-helm).

## How it works

Helm Package Manager has two major components:

### `Helm Client` - command-line client for end users.

- Local chart development
- Interacting with the Tiller server
- Getting release details

### `Tiller` - incluster server that interacts with the Helm client

- Listening for incoming requests from the Helm client  
- Combining a chart and configuration to build a release
- Installing charts into Kubernetes, and then tracking the subsequent release
- Upgrading and uninstalling charts by interacting with Kubernetes

![HelmArch](img/helmArch.png)

## Getting starting with HELM Chart Templating

Before creating Helm Charts, lets go through the Helm Chart directory structure.

```sh
mychart/
  Chart.yaml   # contains a description of the chart
  values.yaml  # This file contains the default values for a chart
  charts/      # contain other charts(subcharts)
  templates/   # template of K8s manifest files
  ...
```

Lets follow offcial [helm documetaion](https://helm.sh/docs/chart_template_guide/#a-starter-chart) for creating first chart.