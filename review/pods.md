# The App Runs as a Pod

> _The atomic unit of kubernetes._
>
> _Time: 5m_

The example below creates one pod, holding just one
container.  The container in turn holds just one server,
your app component.

Since a pod is the fundamental unit of scheduling and
replication in k8s, one has the opportunity to provide
some scheduling information at pod creation time.

### The resources that matter to a pod

To run, a pod needs memory and CPU.  A node knows
how much it has of each:

<!-- @nodeCapacities @test -->
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

Each container in a pod specifies the CPU and memory it
needs via the `resources.requests` and
`resources.limits` variables.

The values assigned to `requests` and `limits` for a
container determine that container's _quality of
service_ class. A pod is given a QoS level matching the
lowest level assigned to any of its containers.

There are three QoS classes:

* _Best-Effort_: `requests` and `limits` not defined,
  lowest QoS.  With no requirements data, k8s
  has no guidance from the container about its needs.
  The pod holding it will be among the first to be
  ejected by the node's kubelet if the node is under
  pressure.  A monitor will see this, and try to
  start the pod somewhere with available resources, but
  will still lack guidance on how to size the job.

* _Burstable_: `requests` < `limits`, medium QoS.
  If its node is under pressure, a pod is ejected if any
  of its containers exceeds their limit and there are
  no _best-effort_ pods around to be sacrificed instead.

* _Guaranteed_: `requests` == `limits`, highest QoS.
  Pod won't be ejected if it doesn't exceed its
  declared limits.  k8s likes these pods because they
  are clear about what they want.  Such a pod may never
  start, because it can be determined in advance that
  no nodes can hold it.  That's better than flapping.
  Values for both `memory` and `cpu` must be specified
  and must respectively match in `request` and
  `limit`.

## Make a pod

In the a pod spec, the `cpu` units are _number of
cores_.  The expression `0.1` is equivalent to the
expression `100m`, which can be read as _one hundred
millicpu_.  The memory units are bytes, with the
suffixes `K`, `M`, `G` etc.

For the capacity exercise below, express the capacities
using bash variables:

<!-- @env @test -->
```
TUT_CON_CPU=100m     # 10% of a CPU
TUT_CON_MEMORY=10000Ki
```

Define a function to create a pod,
and give the pod the label pair _app : tuthello_:

<!-- @funcToCreatePod @env @test -->
```
function tut_createPod {
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: $1
  labels:
    app: tuthello
spec:
  # Pod must have at least one container.
  containers:
    - name: cnt-carrot
      image: $TUT_IMG_REPO:1
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

<!-- @createThePod @test -->
```
tut_createPod pod-tomato
```

This prints the name of all pods:
<!-- @getAllPods @test -->
```
kubectl get pods
```

This describes the name pod in more detail,
report its _QoS_ class, its _Ready_
state, the node it's running on, etc.

<!-- @describeOnePod @test -->
```
kubectl describe pod pod-tomato
```

<!-- @funcDetailPod @env @test -->
```
function tut_detailPod {
  local tmpl=`cat <<EOF
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
  kubectl get -o go-template="$tmpl" pod $1
}
```

<!-- @detailPod @test -->
```
tut_detailPod pod-tomato
```

### scheduling test

To test resource control, try to make
a pod that requires seven CPUS:

<!-- @unschedulablePod -->
```
TUT_CON_CPU=7000m # 7 cpus
tut_createPod pod-turnip
```

A pod configured to use this much CPU is unschedulable,
since no nodes have that many cores available.  Various
jobs (e.g. kubelet, fluentd, etc) consume some
percentage of the cpu.

Verify that the pod has not entered the running phase:

```
tut_detailPod pod-turnip
```

Get rid of it:

<!-- @deletePod -->
```
kubectl delete pod pod-turnip
```
