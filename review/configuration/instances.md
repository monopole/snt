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
The tradeoffs between using labels and namespaces to
partition resources was discussed
[earlier](/review/namespaces) (e.g., as the instance
count increases, using namespaces make less sense).

The deployment and service resources will be almost
identical, so we'll use a cheap template to make them.

### Cheap templating for deployment and service

Write two files.  One will serve as a template for a
deployment, the other as a template for a service.

The image tag is immediately "templated" into the
former via the env var `TUT_IMG_TAG`, to accomodate
minikube vs GKE differences.

<!-- @clearDirectory @test -->
```
/bin/rm -rf $TUT_DIR/raw
mkdir -p $TUT_DIR/raw
```

<!-- @writeDeploymentTemplate @test -->
```
cat <<EOF >$TUT_DIR/raw/dep-instance.yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: tuthello-myInst
spec:
  replicas: 3
  template:
    metadata:
      name: tuthello-myInst
      labels:
        app: tuthello
        instance: myInst
    spec:
      containers:
      - name: cnt-carrot
        image: $TUT_IMG_TAG:imgVersion
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
              name: tuthello-cfgmap
              key: altGreeting
        - name: ENABLE_RISKY
          valueFrom:
            configMapKeyRef:
              name: tuthello-cfgmap
              key: enableRisky
EOF
```

<!-- @writeServiceTemplate @test -->
```
cat <<EOF >$TUT_DIR/raw/svc-instance.yaml
kind: Service
apiVersion: v1
metadata:
  name: tuthello-myInst
spec:
  selector:
    app: tuthello
    instance: myInst
  type: LoadBalancer
  ports:
  - protocol: TCP
    port: 8666
    targetPort: 8080
EOF
```

### Make the configMaps

Write two files, representing _most_ of the differences
between a first release of the app and a second
release.  The remaining diffence is _the version of the
underlying binary_.

These maps are just data, so they aren't given instance
labels.  They do get app labels, since they are data
for `tuthello`.

<!-- @mapForRelease1 @test -->
```
cat <<EOF >$TUT_DIR/raw/cfg-release-1.yaml
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
cat <<EOF >$TUT_DIR/raw/cfg-release-2.yaml
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

For use in concatenating YAML, make a tiny separator file:


<!-- @mapForRelease2 @test -->
```
echo '---' >$TUT_DIR/raw/sep.yaml
```

The resource templates and configMaps are in place:
<!-- @list @test -->
```
ls -C1 $TUT_DIR/raw
```

Next configure the cluster.
