# ConfigMaps

[ConfigMap]: https://kubernetes.io/docs/tasks/configure-pod-container/configmap

Now redo the above deployment, but this time connect
the deployment to a [ConfigMap] and use it
reconfigure the image held by the pods with `kubectl`.

First create the map:

<!-- @createConfigMap -->
```yaml
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: $TUT_CFG_NAME_1
data:
  tutorial.greeting: "bonjour"
  tutorial.enablerisk: "false"
EOF
```

<!-- @describeConfigMap -->
```
kubectl describe configmap $TUT_CFG_NAME_1
```

To illustrate a point made below, change the image's
(container's) port value:

<!-- @tryNonDefaultPortToCheckFlagValuePassing -->
```
TUT_CON_PORT_VALUE=8777
```

and recreate the service (the loadbalancer):

<!-- @deleteAndRecreateService -->
```
kubectl delete service $TUT_SERVICE_NAME
tut_CreateService
```

The loadbalancer, when started, is told which port to
map to on the various nodes.  If the port value is
changed in a deployment (which is about to happen), the
service will not notice - it has to be deleted and
restarted.

Now create a deployment that uses the new container
port value, and references the new configmap:

<!-- @reCreateDeploymentToReferenceConfigMaop -->
```yaml
cat <<EOF | kubectl create -f -
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: $TUT_DEPLOY_NAME
spec:
  replicas: 3
  template:
    metadata:
      name: $TUT_POD_NAME
      labels:
        app: $TUT_APP_LABEL
        env: monkey-staging
    spec:
      containers:
      - name: $TUT_CON_NAME
        image: $TUT_IMG_TAG:$TUT_IMG_V2
        # --port value must match container port, so it's not safe
        # to set it via configmap.  But it's wise safe to specify
        # it explicity in this declaration.
        command: ["/${TUT_IMG_NAME}",
                  "--port=${TUT_CON_PORT_VALUE}",
                  "--enableRiskyFeature=\$(TUTORIAL_ENABLE_RISK)"]
        resources:
          limits:
            cpu: $TUT_CON_CPU
            memory: $TUT_CON_MEMORY
          requests:
            cpu: $TUT_CON_CPU
            memory: $TUT_CON_MEMORY
        ports:
        - name: $TUT_CON_PORT_NAME
          containerPort: $TUT_CON_PORT_VALUE
        env:
        - name: TUTORIAL_GREETING
          valueFrom:
            configMapKeyRef:
              name: $TUT_CFG_NAME_1
              key: tutorial.greeting
        - name: TUTORIAL_ENABLE_RISK
          valueFrom:
            configMapKeyRef:
              name: $TUT_CFG_NAME_1
              key: tutorial.enablerisk
EOF
```

<!-- @describeDeployments -->
```
kubectl describe deployments
```

The following query should work:

<!-- @curlService -->
```
tut_Query lime
```

Apply a change in the configuration that launches the
risky feature and changes the greeting:

<!-- @applyConfigMapChange -->
```yaml
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: $TUT_CFG_NAME_1
data:
  tutorial.greeting: "GREETING ONE :-("
  tutorial.enablerisk: "true"
EOF
```

<!-- @describeConfig -->
```
kubectl describe configmap $TUT_CFG_NAME_1
```

There are no visible changes in the service yet.

<!-- @curlService -->
```
tut_Query lemon
```

Delete one pod at a time and watch as new configuration
is adopted by new pods.

<!-- @deleteOnePod -->
```
tut_DeleteRandomPod
```

Repeat this query a few times to hit different pods:

<!-- @tryQuery -->
```
tut_Query orange
```

One can get the cluster into a (likely very
undesirabled) mixed state by _alternating changing the
config with deletion of a single pod_.

One way to get everything in sync again is delete all
pods to force recreation at the latest config:

<!-- @deleteAllPods -->
```
kubectl delete --all pods
```

The _recommended way_ to manage config is to not apply
changes to a configmap, but to instead create a _new configmap_
and apply a change to the deployment so that it points to
that new configmap.

Create a second config:

<!-- @createConfigMap2 -->
```yaml
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: $TUT_CFG_NAME_2
data:
  tutorial.greeting: "greeting two :-)"
  tutorial.enablerisk: "false"
EOF
```

<!-- @describeConfigMap -->
```
kubectl describe configmap $TUT_CFG_NAME_2
```

Define a function to apply a config change:

<!-- @defineFunctionToPointDeploymentToNewConfig -->
```
function tut_ApplyConfigChange {
local newConfig=$1
echo Changing to config $newConfig
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: $TUT_DEPLOY_NAME
spec:
  template:
    spec:
      containers:
      - name: $TUT_CON_NAME
        env:
        - name: TUTORIAL_GREETING
          valueFrom:
            configMapKeyRef:
              name: $newConfig
              key: tutorial.greeting
        - name: TUTORIAL_ENABLE_RISK
          valueFrom:
            configMapKeyRef:
              name: $newConfig
              key: tutorial.enablerisk
EOF
}
```

Confirm existing behavior before applying the change:
<!-- @curlService -->
```
tut_Query lime
```

Apply the change:

<!-- @changeDeployToConfig2 -->
```
tut_ApplyConfigChange $TUT_CFG_NAME_2
```

Query again, noting the change in the output brought on by the
change in the values of the environment variables.

<!-- @curlService -->
```
tut_Query lime
```

Change it back and forth, and rapidly query to watch the turnover.

<!-- @changeDeployToConfig1 -->
```
tut_ApplyConfigChange $TUT_CFG_NAME_1
for i in {1..15}; do
  tut_Query movingTo1
  sleep 0.5
done
```

<!-- @changeDeployToConfig2 -->
```
tut_ApplyConfigChange $TUT_CFG_NAME_2
for i in {1..15}; do
  tut_Query movingTo2
  sleep 0.5
done
```
