# Instances via patch

In DAM, an _instance_ is yet another manifest and
associated resources, but this time used as an
_overlay_ on the base resource.

Part of it _modifies_
the base resource, and the remainder, in resource form,
_merges_ with the base resource.


<!-- @makePatchManifest -->
```
cat <<'EOF' >$TUT_DAM/instance/staging/Kube-manifest.yaml
apiVersion: manifest.k8s.io/v1alpha1
kind: Package
metadata:
  name: makes-staging-tuthello

description: Tuthello configured for staging

namePrefix: staging-

objectLabels:
  instance: staging
  org: acmeCorporation

objectAnnotations:
  note: Clear is better than clever.

resources:
- ../../manifest

patches:
# - release-1.yaml
# - deployment.yaml
# - service.yaml

EOF
```

<!-- @mapForRelease1 @test -->
```
cat <<EOF >$TUT_DAM/instance/staging/release-1.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mymap
  labels:
    app: tuthello
data:
  altGreeting: "你好"
  enableRisky: "false"
EOF
```

## TODO: Here be dragons

Define a patch, creating a specific instance to merge with base resources.

<!-- @makePatchedDeployment -->
```
cat <<'EOF' >$TUT_DAM/instance/staging/deployment.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: tuthello
spec:
  replicas: 2
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
EOF
```
