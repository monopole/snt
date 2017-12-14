# Helm

> _Configure your cluster using templates._
>
> _Time: 6min_

[Helm](https://helm.sh/) is a relatively mature
means to define, install and upgrade a k8s application,
offered by Microsoft.

## Download

Canonical doc
[here](https://github.com/kubernetes/helm/blob/master/docs/install.md),
shortened to:

<!-- @installHelm -->
```
apis=https://storage.googleapis.com
tarball=helm-v2.7.0-linux-amd64.tar.gz

mkdir -p $TUT_DIR/bin
pushd $TUT_DIR
curl -Lo helm.tgz $apis/kubernetes-helm/$tarball
gunzip <helm.tgz | tar xvf -
rm helm.tgz
mv linux-amd64/helm $TUT_DIR/bin
/bin/rm -rf linux-amd64
popd
alias helm=$TUT_DIR/bin/helm
```

## Installation

Helm installs a service into a cluster,
so you must have one up - i.e.
this command should work:
```
kubectl cluster-info
```

<!-- @initialize -->
```
helm init
```


## Are the lights on?

<!-- @confirmTiller -->
```
kubectl get pods --namespace kube-system | grep tiller
```

```
helm version
```
