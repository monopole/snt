### Try a Deployment


[Deployment]: https://kubernetes.io/docs/concepts/workloads/controllers/deployment
[ServiceSpec]: https://kubernetes.io/docs/api-reference/v1.7/#service-v1-core


Kubernetes adds layers to avoid toil.

* It is toil to repeatedly install all the libraries
  and data needed to make an app work, so one creates a
  _container_ and installs only that.

* It is toil to install a set of containers that
  represent a serving stack every time one is needed,
  so one defines a _pod_ and installs that.

* It is toil to watch and restart pods on unreliable or
  changing hardware, so one invents a _replicaset_ to
  do the job.

* The next level up - a deployment - concerns itself
  with creating replicasets, scaling them, upgrading or
  downgrading the pods therein, etc.

Deployment yaml looks like replicaset yaml.
Only the `kind` changes:

<!-- @createDeployment -->
```yaml
cat <<EOF | kubectl create -f -
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: dep-kale
spec:
  replicas: 2
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
            cpu: $TUT_CON_CPU
            memory: $TUT_CON_MEMORY
          requests:
            cpu: $TUT_CON_CPU
            memory: $TUT_CON_MEMORY
        ports:
        - name: port-pumpkin
          containerPort: 8080
EOF
```


<!-- @getDeployments -->
```
kubectl get deployments
```

<!-- @desribeDeployments -->
```
kubectl describe deployments
```

A deployment differs from a replicaset in that it
manages changes to the replicaset, including creating a
new one while turning down the old one.

Observe this by upgrading the image from v1 to v2.

<!-- @confirmCurrentImageVersion -->
```
kubectl describe pods | grep Image:
tut_Query kiwi
```

Apply a change in the deployment image (to v2):

<!-- @applyUpgrade -->
```yaml
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: dep-kale
spec:
  template:
    spec:
      containers:
      - name: cnt-carrot
        image: $TUT_IMG_TAG:$TUT_IMG_V2
EOF
```

<!-- @checkImageVersion -->
```
kubectl describe pods | grep Image:
tut_Query tangerine
```

When finished playing, delete the deployent.
<!-- @deleteDeployment -->
```
kubectl delete deployment dep-kale
```
