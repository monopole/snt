# Define a Pod

> _The atomic unit of kubernetes._
>
> _Time: 5min_

The example below creates one pod, holding just one
container.  The container in turn holds just one server
binary.

Since a pod is the atomic unit of scheduling and
replication in k8s, one has the opportunity to provide
some scheduling information at pod creation time.

### Capacity is crucial

Each container in a pod can specify the CPU and memory
resources it needs via the `resources.requests` and
`resources.limits` variables.  The values assigned to
`requests` and `limits` for a container determine that
container's _quality of service_ class, aka QoS
class. A pod is given a QoS level matching the lowest
level assigned to any of its containers.

There are three QoS classes:

* _Best-Effort_: `requests` and `limits` not defined,
  lowest QoS.  With no requirements data, the kubelet
  has no guidance from the container about its needs.
  The pod holding it will be among the first to be
  ejected by the node's kubelet if the node is under
  pressure.  The watcher will see this, and try to
  start the pod somewhere with available resources, but
  will still lack guidance on how to size the job.

* _Burstable_: `requests` < `limits`, medium QoS.
  If its node is under pressure, a pod is ejected if any
  of its containers exceeds their limit and there are
  no _best-effort_ pods around to be sacrificed instead.

* _Guaranteed_: `requests` == `limits`, highest QoS.
  Pod won't be ejected if it doesn't exceed its
  declared limits.  The kubelet likes these pods
  because they are clear about what they want.  Such a
  pod may never start, because it can be determined in
  advance that no nodes can hold it - which is better
  than flapping.  Values for both `memory` and `cpu`
  must be specified and must be respectively match in
  `request` and `limit`.

In the spec, the `cpu` units are _number of cores_.
The expression `0.1` is equivalent to the expression
`100m`, which can be read as _one hundred millicpu_.
The memory units are bytes, with the suffixes `K`, `M`,
`G` etc.


<!-- @printNodeCapacities -->
```
tmpl=`cat <<EOF
{{range .items}}
{{.metadata.name -}}
{{with .status}}
    allocatable cpu: {{.allocatable.cpu}}
       capacity cpu: {{.capacity.cpu}}
    allocatable mem: {{.allocatable.memory}}
       capacity mem: {{.capacity.memory}}
{{end}}
{{end}}
EOF
`
kubectl get -o go-template="$tmpl" nodes
```


## Make a pod

<!-- @defineContainerCapacityVarsForDemo -->
```
TUT_CON_CPU=100m     # 10% of a CPU
TUT_CON_MEMORY=10000Ki
```

Define a function to create a pod, do so, then
`get` the pod:

<!-- @defineFunctionToCreatePod-->
```
function tut_CreatePod {
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: pod-tomato
  labels:
    #  Label critical for this example.
    app: avocado
    #  Not used, but including to see it in output.
    env: monkey-staging
spec:
  # Pod must have at least one container.
  containers:
    - name: cnt-carrot
      image: $TUT_IMG_TAG:$TUT_IMG_V1
      imagePullPolicy: IfNotPresent
      resources:
        limits:
          cpu: $TUT_CON_CPU
          memory: $TUT_CON_MEMORY
        requests:
          cpu: $TUT_CON_CPU
          memory: $TUT_CON_MEMORY
      # Any one container can open any number of ports
      ports:
        # This container needs to expose only one port.
        - name: port-pumpkin
        # Specify the default port that the app uses.
          containerPort: 8080
          protocol: TCP
EOF
}
```

<!-- @createThePod -->
```
tut_CreatePod
```

<!-- @getAllPods -->
```
kubectl get pods
```

`Describe` the pod to see the _QoS_ class, the _Ready_
state, the node it's running on, etc.

<!-- @describeOnePod -->
```
kubectl describe pod pod-tomato
```

<!-- @focussedDescribePod -->
```
tmpl=`cat <<EOF
{{.metadata.name -}}
{{range .spec.containers -}}
  {{with .resources}}
      requests cpu: {{.requests.cpu}}
        limits cpu: {{.limits.cpu}}
   requests memory: {{.requests.memory}}
     limits memory: {{.limits.memory}}
  {{- end}}
{{- end -}}
{{with .status}}
               Qos: {{.qosClass}}
             phase: {{.phase}}
  {{- range .conditions -}}
       {{if .message}} Message: {{.message}}{{end -}}
       {{if .reason}} Reason: {{.reason}}{{end -}}
  {{end}}
{{end}}
EOF
`
kubectl get -o go-template="$tmpl" pod pod-tomato
```

### scheduling test

To test resource control, optionally delete the pod and
recreate it after increasing `TUT_CON_CPU` to `1000m`, i.e.:

<!-- @checkScheduling -->
```
kubectl delete pod pod-tomato
sleep 8
TUT_CON_CPU=1000m
tut_CreatePod
```

A pod configured to use 1 (entire) CPU is
unschedulable, since no nodes have 1 cpu available.
Various jobs (e.g. kubelet, fluentd, etc) consume some
percentage of the cpu.

Verify that the pod now suffers from `ContainersNotReady`.

```
kubectl get -o go-template="$tmpl" pod pod-tomato
```

Confirm it can be recreated with a reasonable CPU request:

<!-- @recreatePod -->
```
kubectl delete pod pod-tomato
sleep 8
TUT_CON_CPU=100m
tut_CreatePod
```

<!-- @checkPod -->
```
kubectl get -o go-template="$tmpl" pod pod-tomato
```
