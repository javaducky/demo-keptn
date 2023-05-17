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
Once completed, you should be able to access your service via the ingress at http://localhost:8081. If you chose to include
the Slack integration, you should also have seen notifications in the configured channel.
