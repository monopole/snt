# Resources

> _Make some unadorned resources._
>
> _Time: 1m_

[GCR]: https://cloud.google.com/container-registry/
[helm]: /apps/examples/helm
[baseline]: /apps/examples/baseline

As with the [baseline] and [helm] apps discussed
earlier, resources will be placed in a directory:

<!-- @makeTree @test -->
```
export TUT_DAM=$TUT_DIR/apps/dam/tuthello
/bin/rm -rf $TUT_DAM
mkdir -p $TUT_DAM
```

Again as before, there will be a deployment, a service,
and some configmaps.

The `$TUT_IMG_REPO` variable, replaced on the way to
writing the file, appears here (as [before]) soley to
let this tutorial work with either a local container
registry or [GCR].

Other than that, these resources have no special markup
or names.  Even the resources have uncreative names.

<!-- @writeDeploymentYaml @test -->
```
cat <<EOF >$TUT_DAM/deployment.yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: tut-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        deployment: tuthello
    spec:
      containers:
      - name: cnt-carrot
        image: $TUT_IMG_REPO:1
        command: ["/tuthello",
                  "--port=8080",
                  "--enableRiskyFeature=\$(ENABLE_RISKY)"]
        ports:
        - containerPort: 8080
        env:
        - name: ALT_GREETING
          valueFrom:
            configMapKeyRef:
              name: tut-map
              key: altGreeting
        - name: ENABLE_RISKY
          valueFrom:
            configMapKeyRef:
              name: tut-map
              key: enableRisky
EOF
```

As before, the deployment takes some parameters from a
map:

<!-- @writeMapYaml @test -->
```
cat <<EOF >$TUT_DAM/configMap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: tut-map
data:
  altGreeting: "Good Morning!"
  enableRisky: "false"
EOF
```

Also as before, a service is defined to select pods
associated with the app:

<!-- @writeServiceYaml @test -->
```
cat <<EOF >$TUT_DAM/service.yaml
kind: Service
apiVersion: v1
metadata:
  name: tut-service
spec:
  selector:
    deployment: tuthello
  type: LoadBalancer
  ports:
  - protocol: TCP
    port: 8666
    targetPort: 8080
EOF
```

Review the app definition so far:

<!-- @listFiles @test -->
```
find $TUT_DAM
```
