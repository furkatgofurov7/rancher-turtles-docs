---
sidebar_position: 1
---

# Test suite guide

The main reference for reusing the test suite is [this repository](https://github.com/rancher-sandbox/turtles-integration-suite-example), which contains an example on how to integrate a given CAPI provider with Rancher Turtles and applies a series of checks based on a GitOps workflow.

### Before execution

The end-to-end test environment used in Turtles provides a number of configuration alternatives depending on the type of test you are running and the type of checks you are performing. If getting started with the test suite, we recommend you keep your configuration as simple as possible and limit the number of customizations so you can understand the process and its configuration details. You can start your journey on provider testing by cloning the sample repository:

```
git clone https://github.com/rancher-sandbox/turtles-integration-suite-example.git
```

The simplest test execution you can run creates a local environment that does not use an internet-facing endpoint. This limits the checks to only local downstream clusters (effectively, CAPI clusters provisioned via CAPD) but it is enough to run the example integration. You can simply run this local version by specifying that you intend to run it locally.

```
MANAGEMENT_CLUSTER_ENVIRONMENT="isolated-kind" make test
```

When checking the integration with other infrastructure providers (e.g. providers for cloud vendors), you will have to make your Rancher instance available via endpoint to the downstream clusters, which are no longer in your local environment. The `MANAGEMENT_CLUSTER_ENVIRONMENT` variable we used before, supports the following values:

```
MANAGEMENT_CLUSTER_ENVIRONMENT: "kind" # supported options are eks, isolated-kind, kind
```

`isolated-kind`, which is the value we used for local testing, and `kind` will deploy equivalent local environments. The difference is that `kind` will also configure a publicly accessible endpoint via [ngrok](https://ngrok.com/). You can get a free (limited) `ngrok` endpoint and use it for executing tests. Before running `make test`, you will also need to set the following environment variables:

```
NGROK_API_KEY: ""
NGROK_AUTHTOKEN: ""
```

Using this configuration, during environment creation, the Rancher instance will be configured to be accessible via your `ngrok` endpoint and downstream clusters will be able to communicate with it.

The [Other options](#other-options) section contains more information on what you can configure before execution.

### Basic Workflow

In previous sections we introduced the main actions performed in the sample test integration:

#### Create a management cluster in the desired environment.

This is not a Turtles specific requirement as, when working with CAPI, there needs to be a management cluster that will be used to create resources that represent downstream clusters. This is the main part of the test environment and, depending on the environment variables passed to the test suite, it can either be hosted locally (using `kind`) or in the cloud (`eks`).

#### Install Rancher and Turtles with all prerequisites.

Turtles is a Rancher extension and, as such, it needs a Rancher installation to be deployed. Rancher Manager will be run in the management cluster we created in the first step and the Turtles chart will be installed when Rancher is available. If using an internet-facing configuration, an ingress controller will make Rancher reachable from an outside network (e.g. cluster deployed in the cloud).

#### Run the suite that will create a git repo, apply cluster template using Fleet and verify the cluster is created and successfully imported in Rancher.

The main test suite, and the one used as an example, is based on a GitOps flow and uses [Fleet](https://github.com/rancher/fleet) as a GitOps orchestrator tool. Based on the cluster templates provided (you can check the ones that come with the example integration [here](https://github.com/rancher-sandbox/turtles-integration-suite-example/tree/main/suites/data/cluster-templates)), it will create the CAPI clusters defined in the YAML files. Once this/these cluster/s are available, they will be configured to be [imported into Rancher using Turtles](../../getting-started/create-first-cluster/using_fleet.md) and it will verify that the downstream cluster/s is/are accessible via Rancher. It will also check that deletion can be performed on downstream clusters and that they are no longer available in Rancher.

### Other options

You can take a look at the `config.yaml` [file](https://github.com/rancher-sandbox/turtles-integration-suite-example/blob/main/config/config.yaml) in the `turtles-integration-suite-example` repository, which contains a list of environment variables used during test environment deployment and test execution. The following is a truncated version of the above mentioned YAML file:

```
...
variables:
  CLUSTERCTL_BINARY_PATH: ""
  USE_EXISTING_CLUSTER: "false"
  SKIP_RESOURCE_CLEANUP: "false"
  ARTIFACTS_FOLDER: "_artifacts"
  MANAGEMENT_CLUSTER_ENVIRONMENT: "kind" # supported options are eks, isolated-kind, kind
  RANCHER_VERSION: "v2.8.1"
  KUBERNETES_VERSION: "v1.28.6"
  KUBERNETES_MANAGEMENT_VERSION: "v1.27.0"
  KUBERNETES_MANAGEMENT_AWS_REGION: "eu-west-2"
  RKE2_VERSION: "v1.28.1+rke2r1"
  TURTLES_PATH: "turtles/rancher-turtles"
  TURTLES_REPO_NAME: "turtles"
  TURTLES_URL: https://rancher.github.io/turtles
  TURTLES_VERSION: "v0.10.0"
  RANCHER_HOSTNAME: "localhost"
  RANCHER_FEATURES: ""
  RANCHER_PATH: "rancher-latest/rancher"
  RANCHER_REPO_NAME: "rancher-latest"
  RANCHER_URL: "https://releases.rancher.com/server-charts/latest"
  CERT_MANAGER_URL: "https://charts.jetstack.io"
  CERT_MANAGER_REPO_NAME: "jetstack"
  CERT_MANAGER_PATH: "jetstack/cert-manager"
  ...
  ...
  ...
  HELM_BINARY_PATH: "helm"
  HELM_EXTRA_VALUES_FOLDER: "/tmp"
  # Additional setup for establishing rancher ingress
  NGROK_REPO_NAME: "ngrok"
  NGROK_URL: "https://ngrok.github.io/kubernetes-ingress-controller"
  NGROK_PATH: "ngrok/kubernetes-ingress-controller"
  NGROK_API_KEY: ""
  NGROK_AUTHTOKEN: ""
  GITEA_REPO_NAME: "gitea-charts"
  GITEA_REPO_URL: "https://dl.gitea.com/charts/"
  GITEA_CHART_NAME: "gitea"
  GITEA_CHART_VERSION: "9.4.0"
  ...
```

:::tip
You can refer to [Turtles repository](https://github.com/rancher/turtles/tree/main/test/e2e#e2e-tests) to see all the suites and parameters you can use to customize test execution. We recommend doing this only if you are familiar with the deployment/configuration of the test environment and have specific integration requirements.
:::

