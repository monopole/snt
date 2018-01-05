# Beginning Configuration

[ConfigMap]: https://kubernetes.io/docs/tasks/configure-pod-container/configmap

> _Instead of deleting and recreating a deployment
> to change it, set its properties via a [ConfigMap]._
>
> _Time: 6min_


Create a ConfigMap.  It's a set of key:value pairs.

<!-- @applyConfigMap @test -->
```yaml
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: cfg-parsley
data:
  tutorial.greeting: "bonjour"
  tutorial.enablerisk: "false"
EOF
```

<!-- @descConfigMap @test -->
```
kubectl describe configmap cfg-parsley
```

Create a deployment that uses this configmap:

<!-- @deploymentWithCM @test -->
```yaml
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: dep-kale
spec:
  replicas: 3
  template:
    metadata:
      name: pod-tomato
      labels:
        app: avocado
        env: monkey-staging
    spec:
      containers:
      - name: cnt-carrot
        image: $TUT_IMG_TAG:$TUT_IMG_V2
        # --port value must match containerPort
        command: ["/${TUT_IMG_NAME}",
                  "--port=8080",
                  "--enableRiskyFeature=\$(TUTORIAL_ENABLE_RISK)"]
        resources:
          limits:
            cpu: 100m
            memory: 10Mi
          requests:
            cpu: 100m
            memory: 10Mi
        ports:
        - name: port-pumpkin
          containerPort: 8080
        env:
        - name: TUTORIAL_GREETING
          valueFrom:
            configMapKeyRef:
              name: cfg-parsley
              key: tutorial.greeting
        - name: TUTORIAL_ENABLE_RISK
          valueFrom:
            configMapKeyRef:
              name: cfg-parsley
              key: tutorial.enablerisk
EOF
```

<!-- @descDeployments @test -->
```
kubectl describe deployments
```

Make sure querying works:
<!-- @curlService @test -->
```
tut_query lime
```

Apply a change in the configuration that launches the
risky feature and changes the greeting:

<!-- @applyCMapChange @test -->
```yaml
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: cfg-parsley
data:
  tutorial.greeting: "Bienvenue!"
  tutorial.enablerisk: "true"
EOF
```

<!-- @descConfigMap @test -->
```
kubectl describe configmap cfg-parsley
```

There are no visible changes in the service yet.

<!-- @curlService @test -->
```
tut_query lemon
```

### Mixed states

Delete one pod at a time and watch as new configuration
is adopted by new pods.

<!-- @deleteOnePod @test -->
```
tut_deleteRandomPod
sleep 2
```

Repeat this query ten or so times to hit different pods:

<!-- @tryQuery  -->
```
tut_query orange
```

One can get the cluster into a (presumably
undesirabled) permanently mixed state by changing the
config and deleting a single pod.

Get everything in sync again by deleting all the pods
to force recreation at the latest config:

<!-- @deleteAllPods @test -->
```
kubectl delete --all pods
```

Confirm the new greeting is always served:
<!-- @tryQuery @test -->
```
tut_query orange
```

### Treat configs as immutable

The recommended way to manage config is to not apply
changes to a configmap, but to instead create a _new_
map and apply a change to the deployment so that it
points to that new configmap.

Create a second config:

<!-- @createConfigMap2 @test -->
```yaml
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: cfg-cilantro
data:
  tutorial.greeting: "Irasshaimase!"
  tutorial.enablerisk: "false"
EOF
```

<!-- @descConfigMap @test -->
```
kubectl describe configmap cfg-cilantro
```

Define a function to apply a config change:

<!-- @funcRepointDeployment @env @test -->
```
function tut_applyConfigChange {
local newConfig=$1
echo Changing to config $newConfig
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: dep-kale
spec:
  template:
    metadata:
      name: pod-tomato
      labels:
        app: avocado
        env: monkey-staging
    spec:
      containers:
      - name: cnt-carrot
        image: $TUT_IMG_TAG:$TUT_IMG_V1
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
<!-- @curlService @test -->
```
tut_query lime
```

Apply the change:

<!-- @changeToConfig2 @test -->
```
tut_applyConfigChange cfg-cilantro
sleep 3
```

Query again, noting the change in the output brought on by the
change in the values of the environment variables.

<!-- @curlService @test -->
```
tut_query lime
```

Change it back and forth, and rapidly query to watch the turnover.

<!-- @changeToC1WithQuery @test -->
```
tut_applyConfigChange cfg-parsley
for i in {1..10}; do
  tut_query movingTo1
  sleep 0.5
done
```

<!-- @changeToC2WithQuery @test -->
```
tut_applyConfigChange cfg-cilantro
for i in {1..10}; do
  tut_query movingTo2
  sleep 0.5
done
```

### Cleanup

<!-- @deleteStuff @test -->
```
kubectl delete deployment dep-kale
kubectl delete configmap cfg-parsley
kubectl delete configmap cfg-cilantro
```

### ConfigMaps don't work with everything.

They have a general purpose name, but only deployments
use them.

I.e. one cannot use them to, say, instruct a service
which pod ports to talk to.  If the port used in the
container `command:` line above changed, one would need
a new service configured to look at that port instead.
