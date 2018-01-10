# Helm version of tuthello

> _Prepare two instances of tuthello using helm._
>
> _Time: 4m_

In what preceded, the sample instances differed only in
the names and labels on their resources.  The instances
are otherwise identical, which is useless.

[review]: /review/configuration/instances

The following abandons the sample to focus on
[`tuthello`](/tuthello).

As was done in the [review], two truly different
instances (_staging_ and _production_) will be created.

## Helmification

Helm wants a manifest, templates for the deployment,
service and configMap, some value files, and a
directory to hold it all:

<!-- @makeAppDir @test -->
```
mkdir -p $TUT_DIR/tuthello/templates
```

Next make the manifest, which helm calls a _chart_:

<!-- @makeManifest @test -->
```
cat << EOF >$TUT_DIR/tuthello/Chart.yaml
apiVersion: v1
name: tuthello
description: A server that says hello.
version: 0.1.0
EOF
```

The text for the deployment and service is adapted from
the [review](/review/configuration/instances) and from
the helm [sample](apps/helm/sample.md).  It follows
helm convention, calling the instance a _release_:

<!-- @makeDeploymentTemplate @test -->
```
cat << EOF >$TUT_DIR/tuthello/templates/deployment.yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: {{.Chart.Name}}-{{.Release.Name}}
  labels:
    app: {{ .Chart.Name }}
    chart: {{ .Chart.Name }}-{{ .Chart.Version | replace "+" "_" }}
    release: {{ .Release.Name }}
    heritage: {{ .Release.Service }}
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    metadata:
      labels:
        app: {{.Chart.Name}}
        release: {{ .Release.Name }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
        imagePullPolicy: IfNotPresent
        command: ["/{{.Values.image.repository}}",
                  "--port={{.Values.service.internalPort}}",
                  "--enableRiskyFeature=\$(ENABLE_RISKY)"]
        resources:
          limits:
            cpu: 100m
            memory: 10Mi
          requests:
            cpu: 100m
            memory: 10Mi
        ports:
        - containerPort: {{ .Values.service.internalPort }}
        env:
        - name: ALT_GREETING
          valueFrom:
            configMapKeyRef:
              name: {{.Chart.Name}}-{{.Release.Name}}
              key: altGreeting
        - name: ENABLE_RISKY
          valueFrom:
            configMapKeyRef:
              name: {{.Chart.Name}}-{{.Release.Name}}
              key: enableRisky
EOF
```

<!-- @makeServiceTemplate @test -->
```
cat << EOF >$TUT_DIR/tuthello/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: {{.Chart.Name}}-{{.Release.Name}}
  labels:
    app: {{.Chart.Name}}
    chart: {{ .Chart.Name }}-{{ .Chart.Version | replace "+" "_" }}
    release: {{ .Release.Name }}
spec:
  type: LoadBalancer
  ports:
  - port: {{ .Values.service.externalPort }}
    targetPort: {{ .Values.service.internalPort }}
    protocol: TCP
  selector:
    app: {{.Chart.Name}}
    release: {{ .Release.Name }}
EOF
```

The ConfigMap template will get it's data from helm Values:

<!-- @makeConfigMapTemplate @test -->
```
cat << EOF >$TUT_DIR/tuthello/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{.Chart.Name}}-{{.Release.Name}}
  labels:
    app: {{.Chart.Name}}
    chart: {{ .Chart.Name }}-{{ .Chart.Version | replace "+" "_" }}
    release: {{ .Release.Name }}
data:
  altGreeting: "{{ .Values.altGreeting }}"
  enableRisky: "{{ .Values.enableRisk }}"
EOF
```

Finally, make _three_ values files.

The first will hold information that is the _same_
between the two instances that we nevertheless decided
to extract from the resource template:

<!-- @makeCommonValues @test -->
```
cat << EOF >$TUT_DIR/tuthello/values.yaml
replicaCount: 3
image:
  repository: tuthello
  tag: 1
service:
  name:
  internalPort: 8080
  externalPort: 8666
EOF
```

The remaining two files hold the differences.

This is what we'll use to initialize production:
<!-- @makeProductionValues @test -->
```
cat << EOF >$TUT_DIR/tuthello/release-1.yaml
altGreeting: "你好"
enableRisk: "false"
EOF
```

Staging gets version 2 of the binary, a different greeting,
and enablement of the risky feature:

<!-- @makeStagingValues @test -->
```
cat << EOF >$TUT_DIR/tuthello/release-2.yaml
altGreeting: "Ciao!"
enableRisk: "true"
image:
  tag: 2
EOF
```

Summary:

 * A custom helm app was created, with two values files
   to distinguish and configure instances.
