# kinflate Manifest

> _Create a kinflate application by adding a manifest._
>
> _Time: 1m_

To turn a directory of resources into a kinflate
application, and discover what that means, add a
manifest file.

A kinflate manifest is always called

> `Kube-manifest.yaml`

Make one:

<!-- @defineManifest @demo -->
```
export TUT_DAM_MANIFEST=$TUT_DAM/manifest/Kube-manifest.yaml
```

<!-- @makeManifest @demo -->
```
cat <<'EOF' >$TUT_DAM_MANIFEST

apiVersion: manifest.k8s.io/v1alpha1
kind: Package
metadata:
  name: tuthello

description: Tuthello offers greetings as a service.
keywords: [hospitality, kubernetes]
appVersion: 0.1.0

home: https://github.com/TODO_MakeAHomeForThis

sources:
- https://github.com/TODO_MakeAHomeForThis

icon: https://www.pexels.com/photo/nature-summer-yellow-petals-36728/

maintainers:
- name: Miles Mackentunnel
  email: milesmackentunnel@gmail.com
  github: milesmackentunnel

resources:
- deployment.yaml
- configMap.yaml
- service.yaml

EOF
```

Before going any further, save a copy of the manifest:

<!-- @copyManifest @test -->
```
export TUT_ORG_MANIFEST=$TUT_TMP/original-manifest.yaml
cp $TUT_DAM_MANIFEST $TUT_ORG_MANIFEST
```

## Field discussion

### K8s API fields

[k8s API style]: https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/#required-fields

The manifest follows [k8s API style], defining

 * _apiVersion_ - which version of the k8s API is known to work
   with this object
 * _kind_ - the object type
 * _metadata_ - data that helps uniquely identify the
   object, e.g. a name.

### The Usual Suspects

The manifest has the fields one would expect in an app
manifest - _description_, _version_, _home page_, _maintainers_, etc.

### resources

This field lists the k8s resource definitions one would
expect to find accompanying the manifest, were one to
obtain it _git clone_ or some other method.
