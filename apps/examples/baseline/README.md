# Baseline

> _Define an app as a set of k8s resource files in a directory._
>
> _Time: 2m_

In what follows, the instances are made using nothing
other than posix coreutils, to set a baseline.

### Cheap templating

The deployment and service resources ultimately created
for the different instances will be almost identical,
so we'll make them from common files.

In the deployment, the image repo path is "templated"
into the former via the env var `TUT_IMG_REPO`, to
accomodate minikube vs GKE differences.

The deployment and service defined below are completely
functional - one could immediately `kubectl apply` them
to a cluster.

Make an "app directory" for them:

<!-- @makeAppDir @test -->
```
export TUT_BASELINE=$TUT_DIR/apps/baseline/tuthello
mkdir -p $TUT_BASELINE
```

and create the files:

<!-- @writeDeploymentTemplate @test -->
```
cat <<EOF >$TUT_BASELINE/dep-instance.yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: tuthello-theInst
spec:
  replicas: 3
  template:
    metadata:
      name: tuthello-theInst
      labels:
        app: tuthello
        instance: theInst
    spec:
      containers:
      - name: cnt-carrot
        image: $TUT_IMG_REPO:theImgVersion
        command: ["/tuthello",
                  "--port=8080",
                  "--enableRiskyFeature=\$(ENABLE_RISKY)"]
        ports:
        - containerPort: 8080
        env:
        - name: ALT_GREETING
          valueFrom:
            configMapKeyRef:
              name: tuthello-theConfigMap
              key: altGreeting
        - name: ENABLE_RISKY
          valueFrom:
            configMapKeyRef:
              name: tuthello-theConfigMap
              key: enableRisky
EOF
```

<!-- @writeServiceTemplate @test -->
```
cat <<EOF >$TUT_BASELINE/svc-instance.yaml
kind: Service
apiVersion: v1
metadata:
  name: tuthello-theInst
spec:
  selector:
    app: tuthello
    instance: theInst
  type: LoadBalancer
  ports:
  - protocol: TCP
    port: 8666
    targetPort: 8080
EOF
```

### Make the configMaps

Write two configMap files, representing _most_ of the
difference between a first release of the app and a
second release.

The _remaining_ diffence is the version of the
underlying binary, specified in the `containers.image`
field of the deployment spec, a field not configurable
via a configMap.

These maps are just data, so they aren't given instance
labels.  They do get app labels, since they are data
for `tuthello`.

<!-- @mapForRelease1 @test -->
```
cat <<EOF >$TUT_BASELINE/cfg-release-1.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: tuthello-release-1
  labels:
    app: tuthello
data:
  altGreeting: "你好"
  enableRisky: "false"
EOF
```

<!-- @mapForRelease2 @test -->
```
cat <<EOF >$TUT_BASELINE/cfg-release-2.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: tuthello-release-2
  labels:
    app: tuthello
data:
  altGreeting: "Ciao!"
  enableRisky: "true"
EOF
```

For use in concatenating YAML, make a small separator
file:


<!-- @mapForRelease2 @test -->
```
echo '---' >$TUT_BASELINE/sep.yaml
```

The resource templates and configMaps are in place:
<!-- @list @test -->
```
ls -C1 $TUT_BASELINE
```

Next, deploy the instances to the cluster.
