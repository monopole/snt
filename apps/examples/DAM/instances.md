# Instances

> _Deploying the application as instances._
>
> _Time: 1m_

So far we have only one instance of the app, modified
from the original.

To make more instances, we could copy the entire app
directory (manifest + resources) to new directories and
modify them as desired.  Managing all the configuration
that _the instances have in common_ is now an new
problem.

The DAM approach is to create _overlays_.

An overlay is just a sub-directory with another manifest file,
and optionally more (or fewer) resources.

Part of the overlay _modifies_ the base resources, and the
remainder, in resource form, _merges_ with the base
resources.


## Example

[baseline]: /apps/examples/baseline
[helm]: /apps/examples/helm

As  with [baseline] and [helm], the goal is to create two instances -
_staging_ and _production_.

The differences:

 * In staging we'll enable the risky feature.
 * The greetings will differ.
 * Production will have a higher replica count because
   it takes public traffic.

<!-- @makePatchDirectories @test -->
```
mkdir -p $TUT_DAM/staging
mkdir -p $TUT_DAM/production
```

## Define staging

The enhances the original app manifest with new information
to identify _staging_.  For example, there's a new name prefix.

<!-- @makeStagingManifest @test -->
```
cat <<'EOF' >$TUT_DAM/staging/Kube-manifest.yaml
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
  note: Hello there I am staging!

resources:
- ..

patches:
- map.yaml

EOF
```

The following configmap customization changes the
server greeting from _Good Morning!_ to _Have a
pineapple!_.  It also enables the _risky_ flag.

<!-- @stagingMap @test -->
```
cat <<EOF >$TUT_DAM/staging/map.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: tut-map
data:
  altGreeting: "Have a pineapple!"
  enableRisky: "true"
EOF
```

## Define production

<!-- @makeProductionManifest @test -->
```
cat <<'EOF' >$TUT_DAM/production/Kube-manifest.yaml
apiVersion: manifest.k8s.io/v1alpha1
kind: Package
metadata:
  name: makes-production-tuthello
description: Tuthello configured for production

namePrefix: production-

objectLabels:
  instance: production
  org: acmeCorporation

objectAnnotations:
  note: Hello there I am production!

resources:
- ..

patches:
- deployment.yaml

EOF
```

<!-- @productionDeployment @test -->
```
cat <<EOF >$TUT_DAM/production/deployment.yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: tut-deployment
spec:
  replicas: 6
EOF
```

## Compare them

<!-- @runKinflateStaging @test -->
```
kinflate inflate -f $TUT_DAM/staging
```

<!-- @runKinflateProduction @test -->
```
kinflate inflate -f $TUT_DAM/production
```

<!-- @checkDiffs @test -->
```
diff \
  <(kinflate inflate -f $TUT_DAM/staging) \
  <(kinflate inflate -f $TUT_DAM/production)
```
