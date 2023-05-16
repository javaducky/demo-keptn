# Scenario 1
This will be a basic deployment of an `nginx` server in a new `scenario-1` namespace.
From the base deployment, we'll perform updates to allow updates to the application to be orchestrated by Keptn.

> For the impatient, switch to the `solution` branch to see all the updates for this exercise.

### Step 1: Initial deployment
The `nginx.deployment.yaml` is the basis of our application.
This manifest defines our working `Namespace` and `Deployment`, `Service`, and `Ingress` resources to host our Nginx service.

```shell
kubectl apply -f scenario-1/nginx.deployment.yaml
```
Once completed, you should be able to access your service via the ingress at http://localhost:8081.

As is, Keptn is not involved. We've used the Kubernetes client to deploy directly.

### Step 2: Define the Keptn Application
Now we'll create our first resource specific to Keptn: `KeptnApp`. This defines one or more services (workloads) as a logical group.

Define this new resource by creating the `scenario-1/keptn-app.yaml` file having the following content:
```yaml
apiVersion: lifecycle.keptn.sh/v1alpha3
kind: KeptnApp
metadata:
  name: my-application
  namespace: scenario-1
spec:
  version: "0.1"
  revision: 1
  workloads:
    - name: nginx-deployment
      version: 0.1.0
```

Add the resource to your cluster:
```shell
kubectl apply -f scenario-1/keptn-app.yaml
```

### Step 3: Add labels to Deployment
For the Keptn operators to watch for application updates, we need to provide special labels for any pod instances created by our deployment.
This is done by adding the `app.kubernetes.io/part-of`, `app.kubernetes.io/name`, and `app.kubernetes.io/version` labels.
These labels reference the `KeptApp` we defined in the previous step. 

Keptn allows for the standard Kubernetes labels (recommended) or the Keptn-specific annotations to link the deployment pods to the application.
The following shows the equivalent label/annotations and the field within the KeptnApp definition which the values must match.

| Recommended Label           | Keptn Annotation    | KeptnApp field              |
|-----------------------------|---------------------|-----------------------------|
| `app.kubernetes.io/part-of` | `keptn.sh/app`      | `metadata.name`             |
| `app.kubernetes.io/name`    | `keptn.sh/workload` | `spec.workloads[i].name`    |
| `app.kubernetes.io/version` | `keptn.sh/version`  | `spec.workloads[i].version` |

> :point_right: The Keptn annotation has a higher precedence over the recommended label.

Edit the `nginx.deployment.yaml` manifest, adding the labels to our pod template in the `spec.template.metadata.labels` section of the deployment as in the following snippet:
```yaml
spec:
  template:
    metadata:
      labels:
        app: nginx
        # Adding the following labels for Keptn application
        app.kubernetes.io/part-of: my-application
        app.kubernetes.io/name: nginx-deployment
        app.kubernetes.io/version: 0.1.0
```

Update the resources in your cluster:
```shell
kubectl apply -f scenario-1/nginx.deployment.yaml
```

### Step 4: Create a task definition
Another resource introduced with Keptn is the `KeptnTaskDefinition`. This allows for defining scripts to executed pre- or post-deploy.
We'll create a simple script to dump the JSON values provided to the tasks.

Create `scenario-1/keptn-task-dump-values.yaml` with the following content:
```yaml
apiVersion: lifecycle.keptn.sh/v1alpha3
kind: KeptnTaskDefinition
metadata:
  name: dump-values
  namespace: scenario-1
spec:
  function:
    inline:
      code: |
        const vars = ["CONTEXT","DATA","SECURE_DATA"]
        let value;
        vars.forEach((element) => {
          value = Deno.env.get(element);
          if (value != undefined) {
            console.log(element + ":");
            console.log(value);
          }
        });
```

Update the resource in your cluster:
```shell
kubectl apply -f scenario-1/keptn-task-dump-values.yaml
```

### Step 5: Configure post-deployment task
Connect our newly created task to the post-deployment phase for our application.

Update the deployment resource in the `nginx.deployment.yaml` once again. We'll add another label to have the new task
executed after successfully deploying the application.
Edit the `nginx.deployment.yaml` manifest once again, adding the `keptn.sh/post-deployment-tasks` label to the deployment as in the following snippet:
```yaml
spec:
  template:
    metadata:
      labels:
        app: nginx
        # Adding the following labels for Keptn application
        app.kubernetes.io/part-of: my-application
        app.kubernetes.io/name: nginx-deployment
        app.kubernetes.io/version: 0.1.0
        # Execute the `dump-values` task after deployment
        keptn.sh/post-deployment-tasks: "dump-values"
```

Update the resource in your cluster once again:
```shell
kubectl apply -f scenario-1/keptn-task-dump-values.yaml
```

### Step 6: Enable Keptn orchestration
Even with the annotations, labels, and application defined, Keptn will not act upon resources until it is explicitly enabled.
This is controlled at the `Namespace`-level, using the `keptn.sh/lifecycle-toolkit` annotation.

To enable Keptn for our `scenario-1` namespace, modify the `nginx.deployment.yaml` adding the annotation as in the following snippet:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: scenario-1
  annotations:
    keptn.sh/lifecycle-toolkit: "enabled"
```

### Step 7: Perform a deployment update
For our example, we'll simulate an upgrade to our nginx service. We'll do this by updating the value for the workload version
and `app.kubernetes.io/version` label in the `keptn-app.yaml` and `nginx.deployment.yaml` resources, respectively.

Push the changes to both files using the following:
```shell
kubectl apply -f scenario-1/nginx.deployment-solution.yaml -f scenario-1/keptn-app.yaml 
```

Watching the pods available in the `scenario-1` namespace should activity as the deployment is being rolled out.
Additionally, a pod like `klc-post-dump-values-...` should be created with the results of your task. Drilling into 
the pod logs should show the values from the script.
