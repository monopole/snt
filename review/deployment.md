# Deployments

> _App rollout and rollback._
>
> _Time: 6m_


[_deployment_]: https://kubernetes.io/docs/concepts/workloads/controllers/deployment
[ServiceSpec]: https://kubernetes.io/docs/api-reference/v1.7/#service-v1-core
[_replicaset_]: https://kubernetes.io/docs/concepts/workloads/controllers/replicaset

Kubernetes adds layers to avoid toil.

* It's toil to repeatedly install all the libraries
  and data needed to make a program work, so one creates a
  _container_ and installs only that.

* It's toil to install a set of containers that
  represent a serving stack every time one is needed,
  so one defines a _pod_ and installs that.

* It's toil to watch and restart pods on unreliable or
  changing hardware, so one invents a [_replicaset_] to
  do the job.

* It's toil create replicasets, scale them, upgrade or
  downgrading the pods therein, etc., so one invents a
  [_deployment_] with those concepts built in.

## A pod and a count

A deployment is a pod template, a pod instance count,
and the machinery to manages these instances.  If fewer
instances are found, k8s starts some.  If more are
found, k8s stops them.  There's a notion of managing
configuration changes n pods at a time.

Create some deployment YAML, putting it on
disk for later editting, instead of piping it directly
to `kubectl apply`:

<!-- @deploymentYaml @test -->
```
mkdir -p $TUT_DIR/raw
cat <<EOF >$TUT_DIR/raw/dep-kale.yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: dep-kale
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: tuthello
    spec:
      containers:
      - name: cnt-carrot
        image: $TUT_IMG_TAG:1
        resources:
          limits:
            cpu: 100m
            memory: 10Mi
          requests:
            cpu: 100m
            memory: 10Mi
        ports:
        - containerPort: 8080
EOF
```

Optionally assure that there are no leftover
pods and that queries are therefore failing:
<!-- @confirmNoPods -->
```
kubectl delete --all pods
tut_query svc-eggplant kiwi
```

Create the deployment:

<!-- @createDeployment @test -->
```
cat $TUT_DIR/raw/dep-kale.yaml | kubectl apply -f -
```

Queries should work (again):

<!-- @queryIt @test -->
```
tut_query svc-eggplant kiwi
```

Moreover, there are three pods serving them:

<!-- @getPods @test -->
```
kubectl get pods
```

Now "upgrade" the app, by changing the deployment's
image from version 1 of tuthello to version 2:

<!-- @applyUpgrade @test -->
```
cat $TUT_DIR/raw/dep-kale.yaml |\
    sed 's/:1/:2/' |kubectl apply -f -
```

Optionally watch the rollout.  Queries are still served
at this time, but they could hit an old or new image.

<!-- @watchRollout @test -->
```
kubectl rollout status deployment dep-kale
```

Confirm the version change (it's in the HTML of the response):
<!-- @queryService @test -->
```
tut_query svc-eggplant tangerine
```

The `describe` commands lists events in the deployment lifecycle.
One can see how two replicasets were involved in the upgrade -
one growing, while the other shrank:

<!-- @descDeployments @test -->
```
kubectl describe deployment dep-kale
```

Delete the deployment.
<!-- @deleteDeployment @test -->
```
kubectl delete deployment dep-kale
```

## Other pod managers

Some conceptual siblings to a deployment, in that they
wrap one or more pods, are briefly mentioned below.

[Job]: https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion
[Daemon Set]: https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/
[Stateful Set]: https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/

* A pod intended to do some job and then die, e.g.
  simply perform a git clone, is called a [Job].  The
  job manager will retry if the job doesn't signal
  success.

* A pod intended to run on each node outside the scope
  of normal pod scheduling, e.g. to assure logs
  collection on _every_ node, is part of a [Daemon
  Set].

* Pods that retains state, rather than offloading all
  state to cookies, a database or whatever, and need
  this state to be retained across pod rescheduling are
  part of a [Stateful Set].
