# ConfigMaps

[ConfigMap]: https://kubernetes.io/docs/tasks/configure-pod-container/configmap

> _Change properties of a deployment via a [ConfigMap],
> to avoid having to re-specify send configuration
> values into a deployment.  Easier than deleting and
> recreating a deployment._
>
> _Time: 5min_


Create a [ConfigMap]:

<!-- @createConfigMap @test -->
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

<!-- @describeConfigMap @test -->
```
kubectl describe configmap cfg-parsley
```

Create a deployment that uses this configmap:

<!-- @createDeploymentToReferenceConfigMap @test -->
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

<!-- @describeDeployments -->
```
kubectl describe deployments
```

Make sure querying works:
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
  name: cfg-parsley
data:
  tutorial.greeting: "Bienvenue!"
  tutorial.enablerisk: "true"
EOF
```

<!-- @describeConfig -->
```
kubectl describe configmap cfg-parsley
```

There are no visible changes in the service yet.

<!-- @curlService -->
```
tut_Query lemon
```

### Mixed states

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

One can get the cluster into a (presumably
undesirabled) permanently mixed state by changing the
config and deleting a single pod.

Get everything in sync again by deleting all the pods
to force recreation at the latest config:

<!-- @deleteAllPods -->
```
kubectl delete --all pods
```

Confirm the new greeting is always served:
<!-- @tryQuery -->
```
tut_Query orange
```

### Treat configs as immutable

The recommended way to manage config is to not apply
changes to a configmap, but to instead create a _new_
map and apply a change to the deployment so that it
points to that new configmap.

Create a second config:

<!-- @createConfigMap2 -->
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

<!-- @describeConfigMap -->
```
kubectl describe configmap cfg-cilantro
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
<!-- @curlService -->
```
tut_Query lime
```

Apply the change:

<!-- @changeDeployToConfig2 -->
```
tut_ApplyConfigChange cfg-cilantro
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
tut_ApplyConfigChange cfg-parsley
for i in {1..15}; do
  tut_Query movingTo1
  sleep 0.5
done
```

<!-- @changeDeployToConfig2 -->
```
tut_ApplyConfigChange cfg-cilantro
for i in {1..15}; do
  tut_Query movingTo2
  sleep 0.5
done
```

-----

### ConfigMaps don't work with servers.

It's tempting to define the container port in a
configmap, as it's a key part of deploying services.

However, the service resource doesn't yet honor
the notion of a config map.  I.e., one could specify
a port environment variable in a configmap,
and use it in the `command:` field in the deployment's
container spec, but to benefit from this the load
balancer in front of the deployment should _also_
get port information from the same config map.
That currently doesn't work.

<!-- @curlService -->
```
tut_Query lime
```

That's because the service (load balancer) being used
was defined using `targetPort: 8080`:

<!-- @checkServiceTargetPort -->
```
kubectl describe service svc-eggplant | grep TargetPort
```

while the deployment defined above uses port 8777.


done to allow the following discussion:

Suppose we change the service
We could change that with an `apply`:

<!-- @changeTheServicePort -->
```
cat <<EOF | kubectl apply -f -
kind: Service
apiVersion: v1
metadata:
  name: svc-eggplant
spec:
  type: LoadBalancer
  ports:
    - port: 8088
      targetPort: 8777
EOF
```

But the query still doesn't work
<!-- @curlService -->
```
tut_Query lime
```
because (presumably) the apply doesn't trigger a rescan
of the pods to hook up to the actual, virtual cluster ports
in use.

The point being made here is that ports

and recreate the service (the loadbalancer):

<!-- @deleteAndRecreateService -->
```
kubectl delete service svc-eggplant
tut_CreateService
```


```
kubectl describe service svc-eggplant
TUT_SVC_ADDRESS=$(tut_getServiceAddress)
echo "Service at $TUT_SVC_ADDRESS"
```

The loadbalancer, when started, is told which port to
map to on the various nodes.  If the port value is
changed in a deployment (which is about to happen), the
service will not notice - it has to be deleted and
restarted.


---
