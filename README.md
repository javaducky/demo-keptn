# Demo of Keptn

Demonstration project for the ["Creating Reliability Pipelines With Keptn Quality Gates"](https://www.meetup.com/kubernetes-cloud-native-stl/events/292339663/)
([Video](https://www.youtube.com/watch?v=5u6GJIU0oi8))talk, originally presented to the _Kubernetes & Cloud Native STL_
meetup group.

> ### Please don't be confused by the original title.
> 
> The original concept for the talk was to cover what is now known as [Keptn v1](https://keptn.sh/). Upon researching the topic, it was decided
> to pivot to the future of Keptn, i.e. the [Keptn Lifecycle Toolkit (KLT)](https://lifecycle.keptn.sh/). The KLT is considered the cloud native
> approach for orchestrating deployments with Keptn and is the direction of the project going forward.

## Prerequisites
* [git](https://git-scm.com/) - For accessing the sourcecode repositories.
* [Docker](https://docs.docker.com/get-docker/) - For building our custom k6 and running the examples.
* [kubectl](https://kubernetes.io/releases/download/#kubectl) - Client for talking to Kubernetes clusters.
* [go](https://go.dev/doc/install) - To build and install k6-operator.
* TODO I may not use this... [Yq](https://mikefarah.gitbook.io/yq/) - Just for the helper scripts to parse YAML files.

There _may_ be others that I didn't recall as having them installed long ago. My apologies for any issues!

## Create a local Kubernetes cluster
I'm using [k3d](https://k3d.io/) locally to run a _Kubernetes_ cluster within _Docker_. 

> :point_right: You can use any other Kubernetes cluster at your disposal as long as it is version 1.24 or greater.

Once installed k3d has been installed, I use the following command to create a cluster named `keptn-demo-cluster`.

```shell
k3d cluster create keptn-demo-cluster \
 --api-port 6550 \
 -p "8081:80@loadbalancer" \
 --agents 3 \
 --k3s-arg '--kube-apiserver-arg=feature-gates=EphemeralContainers=true@server:*'
```
> :point_right: If you've previously created the cluster, you can start the cluster using `k3d cluster start keptn-demo-cluster`
> if not already running.

Once this is complete, I now have a running Kubernetes cluster on which I can use `kubectl` as well as other tooling
like [k9s](https://k9scli.io/).

## Installing the Keptn Lifecycle Toolkit
For this demonstration, we're going to install using the manifest for the latest release. Other [installation methods](https://lifecycle.keptn.sh/docs/install/),
including Helm, are available in the Keptn documentation.

> :question: _Why are we deploying via manifest?_
> 
> For use in Docker, we're wanting to adjust some of the resource requests given limited resources.

1. First, let's download latest manifest file from releases (https://github.com/keptn/lifecycle-toolkit/releases).
I'll be saving this in my project root directory as `manifest.yaml`.
1. Modify the `manifest.yaml`, changing the CPU request for the scheduler component. We'll change the resource request from `cpu: 100m` to `cpu: 10m`. (line 4336 for v0.7.1)
1. Create a namespace for our toolkit installation:
   ```shell
   kubectl create ns keptn-lifecycle-toolkit-system
   ```
1. Now finish our installation by applying the updated manifest:
   ```shell
   kubectl apply -f manifest.yaml
   ```
Using a tool like [k9s](https://k9scli.io/), you can watch for the installation to complete. You should now see that your
cluster has the `certificate-operator`, `lifecycle-operator`, `metrics-operator`, and `scheduler-operator` all running.

## Scenarios
We'll walk through multiple examples using the lifecycle toolkit. All scenarios will depend upon the preceding installation step.

| Scenario                  | Description                                                                |
|---------------------------|----------------------------------------------------------------------------|
| [scenario-1](scenario-1/) | Enable Keptn orchestration for a basic application with a post-deploy task |
| [scenario-2](scenario-2/) | Work with an application having multiple services                          |

## Resources
- https://github.com/keptn/lifecycle-toolkit
