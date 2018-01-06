# Container Configuration

[ConfigMap]: https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap

> _A native means to make sharable sets of key:value
> pairs to configure containers._
>
> _Total Time: 10m_

So far the tutorial has just run the `tuthello` server
without overriding the default values of environment
variables or command line arguments.

The following inject values into the container via a
[ConfigMap]; it's a named set of key:value pairs.

Create a map called _cfg-parsley_ to continue the
pattern of using _resourceHint-vegetable_ to make names
stand out from overused terms like service, release, etc.

<!-- @applyConfigMap @test -->
```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: cfg-parsley
data:
  altGreeting: "Bonjour"
  enableRisky: "false"
EOF
```

<!-- @descConfigMap @test -->
```
kubectl describe configmap cfg-parsley
```

Modify the deployment so that it
uses `cfg-parsley`:

<!-- @writeDepWithMap @test -->
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
        # --port value must match containerPort
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
              name: cfg-parsley
              key: altGreeting
        - name: ENABLE_RISKY
          valueFrom:
            configMapKeyRef:
              name: cfg-parsley
              key: enableRisky
EOF
```

Create it:
<!-- @createDep @test -->
```
cat $TUT_DIR/raw/dep-kale.yaml | kubectl apply -f -
```

Describe it to see the new _Environment_ section:
<!-- @descDep @test -->
```
kubectl describe deployment dep-kale
```

Confirm that _Bonjour_ has replaced the default
greeting of _Hello_:

<!-- @curlService @test -->
```
tut_query svc-eggplant lime | grep Bonjour
```
