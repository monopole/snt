# App Rollout and Rollback

> _Rollout a new server._
>
> _Time: 6min_


[Deployment]: https://kubernetes.io/docs/concepts/workloads/controllers/deployment
[ServiceSpec]: https://kubernetes.io/docs/api-reference/v1.7/#service-v1-core


Kubernetes adds layers to avoid toil.

* It's toil to repeatedly install all the libraries
  and data needed to make a program work, so one creates a
  _container_ and installs only that.

* It's toil to install a set of containers that
  represent a serving stack every time one is needed,
  so one defines a _pod_ and installs that.

* It's toil to watch and restart pods on unreliable or
  changing hardware, so one invents a _replicaset_ to
  do the job.

* It's toil create replicasets, scale them, upgrade or
  downgrading the pods therein, etc., so one invents a
  _deployment_ with those concepts built in.

Deployment yaml looks like replicaset yaml.
Only the `kind` changes:

<!-- @createDeployment @test -->
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
        image: $TUT_IMG_TAG:$TUT_IMG_V1
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
EOF
```

<!-- @getDeployments @test -->
```
kubectl get deployments
```

<!-- @descDeployments @test -->
```
kubectl describe deployments
```

A deployment contains a replicaset. The cluster now has
one, with a generated name built from the deployment
name:

<!-- @getReplicaSets @test -->
```
kubectl get replicasets
```

A deployment differs from a replicaset in that it
manages changes to the replicaset, including creating a
new one while turning down the old one.

Observe this by upgrading the image from v1 to v2.

<!-- @checkVersion @test -->
```
kubectl describe pods | egrep '(Status:|Image:)'
tut_Query kiwi
```

Apply a change in the deployment image (to v2):

<!-- @applyUpgrade @test -->
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
      labels:
        app: avocado
        env: monkey-staging
    spec:
      containers:
      - name: cnt-carrot
        image: $TUT_IMG_TAG:$TUT_IMG_V2
EOF
```

Try the following repeatedly to watch the pod count
change as the new image rolls out:

<!-- @checkAgain @test -->
```
kubectl describe pods | egrep '(Status:|Image:)'
sleep 3
kubectl describe pods | egrep '(Status:|Image:)'
sleep 3
kubectl describe pods | egrep '(Status:|Image:)'
```

The service still works during this process:
<!-- @queryService @test -->
```
tut_Query tangerine
```

When finished, delete the deployent.
<!-- @deleteDeployment @test -->
```
kubectl delete deployment dep-kale
```
