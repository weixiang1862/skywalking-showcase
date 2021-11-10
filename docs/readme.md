# SkyWalking showcase

This showcase repository includes an example music application and other manifests to demonstrate the main features of
SkyWalking. The music application is composed of several microservices that are written in different programming
languages. Here is the architecture:

```mermaid
%% please read this doc in our official website, otherwise the graph is not correctly rendered. 
graph LR;
  loadgen[load generator] --> ui("UI (React)") --> app("app server (NodeJS)") --> gateway("gateway (Spring)");
  gateway --> songs("songs (Spring)") & rcmd("recommendations (Python)");
  rcmd --> songs;
  songs --> db("database (H2)");
```

## Usage

The showcase uses [GNU Make](https://www.gnu.org/software/make/) and Docker containers to run commands, so please make
sure you have `make` installed and Docker daemon running.

### Customization

The variables defined in [`Makefile.in`](../Makefile.in) can be overridden to customize the showcase, by specifying an
environment variable with the same name, e.g.:

```shell
export ES_VERSION=7.14.0
make <target>
```

or directly specifying in the `make` command, e.g.: `make <target> ES_VERSION=7.14.0`.

Run `make help` to get more information.

### Features

The showcase is composed of a set of scenarios with feature flags, you can deploy some of them that interest you by
overriding the `FEATURE_FLAGS` variable defined in [`Makefile.in`](../Makefile.in), as documented
in [Customization](#customization), e.g.:

```shell
make deploy.kubernetes FEATURE_FLAGS=single-node,agent
```

Feature flags for different platforms ([Kubernetes](#kubernetes) and [Docker Compose](#docker-compose)) are not
necessarily the same so make sure to specify the right feature flags.

Currently, the features supported are:

| Name          | Description | Note |
| -----------   | ----------- | ----------- |
| `agent`       | Deploy microservices with SkyWalking agent enabled. | The microservices include agents for Java, NodeJS server, browser, Python. |
| `cluster`     | Deploy SkyWalking OAP in cluster mode, with 2 nodes, and SkyWalking RocketBot UI, ElasticSearch as storage. | Only one of `cluster` or `single-node` can be enabled. |
| `single-node` | Deploy only one single node of SkyWalking OAP, and SkyWalking RocketBot UI, ElasticSearch as storage. | Only one of `cluster` or `single-node` can be enabled. |
| `so11y`       | Enable SkyWalking self observability. | This is enabled by default for platform [Docker Compose](#docker-compose). |
| `vm`          | Start 2 virtual machines and export their metrics to SkyWalking. | The "virtual machines" are mimicked by Docker containers or Pods. |
| `als`         | Start microservices **
WITHOUT** SkyWalking agent enabled, and configure SkyWalking to analyze the topology and metrics from their access logs. | Command `istioctl` is required to run this feature. The agentless microservices will be running at namespace `${NAMESPACE}-agentless` |

### Kubernetes

To deploy the example application in Kubernetes, please make sure that you have `kubectl` command available, and it can
connect to the Kubernetes cluster successfully.

If you don't have a running cluster, you can also leverage [KinD (Kubernetes in Docker)](https://kind.sigs.k8s.io)
or [minikube](https://minikube.sigs.k8s.io) to create a cluster.

Run `kubectl get nodes` to check the connectivity before going to next step. The typical error message that indicates
your `kubectl` cannot connect to a cluster is:

```text
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

#### Deploy

```shell
# Set the features that you want to deploy
export FEATURE_FLAGS=cluster,agent,vm,so11y
# Deploy
make deploy.kubernetes
# Undeploy
make undeploy.kubernetes
# Redeploy
make redeploy.kubernetes # equivalent to make undeploy.kubernetes deploy.kubernetes
```

### Docker Compose

#### Deploy

```shell
# Set the features that you want to deploy
export FEATURE_FLAGS=cluster,agent,vm,so11y
# Deploy
make deploy.docker
# Undeploy
make undeploy.docker
# Redeploy
make redeploy.docker # equivalent to make undeploy.docker deploy.docker
```