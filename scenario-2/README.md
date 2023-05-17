# Scenario 2
_Greatly inspired by https://github.com/keptn/lifecycle-toolkit/blob/main/examples/sample-app/README.md._

This scenario starts with Keptn-managed deployment of the [`podtato-head`](https://github.com/podtato-head/podtato-head-app) 
microservice project in a new `scenario-2` namespace. From this base deployment, we'll perform updates to allow Keptn
to connect to your observability backend for metrics.

> For the impatient, switch to the `solution` branch to see all the updates for this exercise.

### Step 1: Integrate Slack notifications (Optional)
Follow the _Create Slack Webhook_ and _Create slack-secret_ sections in the [PostDeploymentSlackNotification](https://github.com/keptn/lifecycle-toolkit/blob/main/examples/sample-app/README.md#postdeployment-slack-notification)
instructions from the Keptn project. This will create the `slack-secret.yaml` resource which will be used in 
the following step to send notifications to a Slack channel as deployments take place.

### Step 2: Initial deployment
The `podtato-head-manifest.yaml` is the bulk of our application. This manifest defines our working `Namespace`, `Deployment`, 
`Service`, and `Ingress` resources to host our application. The `keptn-app.yaml` and `keptn-task-notify-slack.yaml` define
Keptn resources for the application and post-deploy tasks respectively.

```shell
kubectl apply -f scenario-2/podtato-head-manifest.yaml \
 -f scenario-2/keptn-app.yaml \
 -f scenario-2/keptn-task-notify-slack.yaml \
 -f scenario-2/slack-secret.yaml
```

You can watch the updates using the following custom resources:
```shell
kubectl get keptnworkloadinstances -n scenario-2 -w
```
and
```shell
kubectl get keptnappversions -n scenario-2 -w
```

Once completed, you should be able to access your service via the ingress at http://localhost:8081. If you chose to include
the Slack integration, you should also have seen notifications in the configured channel.

### Step 3: Adding a Pre-Deploy Task
Starting up each application is somewhat of a race. One service may spin up quicker than another. To control this a bit, 
let's add pre-deploy tasks to the _body part_ services so that they aren't started until the frontend display is available.

> :point_up: Another use could be for a service to wait for, or ensure, a database is available.

Create a new `KeptnTaskDefinition` as `scenario-2/keptn-task-check-frontend.yaml` with the following content:
```yaml
apiVersion: lifecycle.keptn.sh/v1alpha3
kind: KeptnTaskDefinition
metadata:
  name: check-frontend-available
  namespace: scenario-2
spec:
  function:
    httpRef:
      # Let's reuse the simple script available in the Keptn samples.
      url: https://raw.githubusercontent.com/keptn/lifecycle-toolkit/main/functions-runtime/samples/ts/http.ts
    parameters:
      map:
        url: http://podtato-head-frontend.scenario-2.svc.cluster.local:8080
```
Next we update the `scenario-2/podtato-head-manifest.yaml` to include the task for each of the _body part_ deployments.
Edit the manifest, bumping the version number to `0.1.1` and adding the labels to our pod template in the `spec.template.metadata.labels` 
section of each deployment as in the following snippet:
```yaml
spec:
  ...
  template:
    metadata:
      labels:
        ...
        app.kubernetes.io/part-of: podtato-head
        app.kubernetes.io/version: 0.1.1
        keptn.sh/pre-deployment-tasks: check-frontend-available
        ...
```
Update the `keptn-app.yaml` file to match the new version as in the following snippet:
```yaml
...
spec:
  version: "1.0.0"
  workloads:
    - name: podtato-head-left-arm
      version: 0.1.1
    - name: podtato-head-frontend
      version: 0.1.0
    ...
  postDeploymentTasks:
    - notify-slack
```

> :point_right: We're not able to simply add an application-wide `preDeploymentTasks` entry fot the frontend check because the frontend cannot wait for itself.

Update the resources in your cluster:

```shell
kubectl apply -f scenario-2/keptn-app.yaml -f scenario-2/keptn-task-check-frontend.yaml -f scenario-2/podtato-head-manifest.yaml
```
