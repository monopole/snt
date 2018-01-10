# Resources

The resources delivered with the manifest are nearly
identical to those used earlier to define a so-called
raw k8s app.

As in the raw example, the resources are fully
functional types - one can immediately apply them to a
cluster to create working services.

The difference is

 * The file names are even more generic. The name
   matches the kind - e.g. a _deployment_ is defined in
   a file called `deployment.yaml`.  The file names
   could be anything, e.g. vegetables, but its simplest
   to match the kinds.

 * The values in the file lack special strings
   (_theInst_, _theConfigMap_, _theImgVersion_) that
   were used before as _sed_ replacement targets to
   generate the instances.

[GCR]: https://cloud.google.com/container-registry/

The remarkable thing is that there's nothing
remarkable.  The `$TUT_IMG_REPO` variable, replaced on
the way to writing the file, appears here (as before)
soley to let this let this tutorial work with either a local
container registry or [GCR].

<!-- @writeDeploymentTemplate @test -->
```
cat <<EOF >$TUT_DAM/manifest/deployment.yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: tuthello
  labels:
    app: tuthello
spec:
  replicas: 3
  template:
    metadata:
      name: tuthello
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
cat <<EOF >$TUT_DAM/manifest/service.yaml
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
