# App Instances

> _Create multiple instances of the app in the cluster._
>
> _Time: 1m_

The pieces are in place to intentionally have different
instances of `tutHello` co-existing in the cluster at
the same time, each running with their own sets of
pods.

Scenario:

 * Create an instance called _production_.
 * Create a different instance called _staging_.
 * Once QA completes on _staging_, upgrade _production_
   to match it.  This is the rollout.
 * Users report a bug in production. Do a rollback.

Subsequent steps (_add a test to repro the bug, change
code to make test pass, push to staging, get QA
approval, push to prod, then continue the cycle with a
new release to staging, etc._) won't be done, as no new
concept is needed.

## Ingredients

Each instance will need

 * a deployment resource,
 * a service resource (to tie a port to labelled pods),
 * configuration - the container image to run, the
   configMap resource to use, and whatever else needs
   to change in the deployment.

Labels will be used to partition instance resources.
The differences between using labels and namespaces to
partition resources was discussed
[earlier](/review/namespaces).

The deployment and service resources will be almost
identical, so we'll use a cheap template to make them.

### Cheap templating for deployment and service

Wipe the app directory:

<!-- @wipeAppDir @test -->
```
/bin/rm $TUT_APP_DIR/*
```

Now write the deployment and service resource files.

The image repo path is "templated" into the
former via the env var `TUT_IMG_REPO`, to accomodate
minikube vs GKE differences.

The deployment and service defined below are completely
functional - one could immediately `kubectl apply` them
to a cluster.

<!-- @writeDeploymentTemplate @test -->
```
cat <<EOF >$TUT_APP_DIR/dep-instance.yaml
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
        resources:
          limits:
            cpu: 100m
            memory: 10Mi
          requests:
            cpu: 100m
            memory: 10Mi
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
cat <<EOF >$TUT_APP_DIR/svc-instance.yaml
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

Write two files (one configMap to each), representing
most of the difference between a first release of the
app and a second release.

The remaining diffence is _the version of the
underlying binary_, specified in the
`containers.image:` field of the deployment spec, a
field not configurable via a configMap.

These maps are just data, so they aren't given instance
labels.  They do get app labels, since they are data
for `tuthello`.

<!-- @mapForRelease1 @test -->
```
cat <<EOF >$TUT_APP_DIR/cfg-release-1.yaml
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
cat <<EOF >$TUT_APP_DIR/cfg-release-2.yaml
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
echo '---' >$TUT_APP_DIR/sep.yaml
```

The resource templates and configMaps are in place:
<!-- @list @test -->
```
ls -C1 $TUT_APP_DIR
```

Next configure the cluster.
