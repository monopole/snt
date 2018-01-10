# kinflate app format

> _Start creating a manifest for the tuthello app._
>
> _Time: 1m_

A kinflate application is described by YAML files arranged
in a particular directory structure:

<!-- @makeTree @test -->
```
export TUT_DAM=$TUT_DIR/tuthello_dam
mkdir -p $TUT_DAM/manifest
mkdir -p $TUT_DAM/instance/staging
mkdir -p $TUT_DAM/instance/production
```

The top level directory, here called `tuthello_dam`, contains the subdirectories
`manifest` and `instances`.

The manifest directory contains the manifest file with the fixed name
`Kube-manifest.yaml`.  It has the fields one would expect in a manifest -
description, version, maintainer, etc.

[k8s API style]: https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/#required-fields

The file also follows [k8s API style], defining

 * apiVersion - which version of the k8s API is used to create this object
 * kind - the object type
 * metadata - data that helps uniquely identify the object, e.g. a name.


<!-- @makeManifest @demo -->
```
cat <<'EOF' >$TUT_DAM/manifest/Kube-manifest.yaml

apiVersion: manifest.k8s.io/v1alpha1
kind: Package
metadata:
  name: tuthello

description: Mungegithub package
keywords: [github, bot, kubernetes]
appVersion: 0.1.0

home: https://github.com/monopole/snt

sources:
- https://github.com/monopole/snt

icon: https://www.pexels.com/photo/nature-summer-yellow-petals-36728/

maintainers:
- name: Miles Mackentunnel
  email: milesmackentunnel@gmail.com
  github: milesmackentunnel

resources:
- deployment.yaml

EOF
```

Next to this file (as siblings in the `manifest` directory) are the app's resources.
