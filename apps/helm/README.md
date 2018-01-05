# Helm

> _Configure your cluster using Go templates._
>
> _Time: 6min_

[Helm](https://helm.sh/) is a relatively mature means
to manage app instances via template substitution.

## Install to disk

[install doc]: https://github.com/kubernetes/helm/blob/master/docs/install.md

Try the following (or visit the canonical [install doc]):

<!-- @installHelm @test -->
```
apis=https://storage.googleapis.com
tarball=helm-v2.7.0-linux-amd64.tar.gz

mkdir -p $TUT_DIR/bin
pushd $TUT_DIR
curl --fail --location --silent \
    -o helm.tgz $apis/kubernetes-helm/$tarball
gunzip <helm.tgz | tar xvf -
rm helm.tgz
mv linux-amd64/helm $TUT_DIR/bin
/bin/rm -rf linux-amd64
popd
alias helm=$TUT_DIR/bin/helm
```

## Install in the cluster

To work, helm must install a service called `tiller`
into the cluster.

Confirm that the cluster started earlier in the
tutorial is still up:

<!-- @isTheClusterUp @test -->
```
kubectl cluster-info
```

Then initialize helm:

<!-- @initialize @test -->
```
helm init
sleep 2  # init returns before completion
```

## Are the lights on?

<!-- @confirmTiller @test -->
```
kubectl get pods --namespace kube-system |\
    grep tiller
```

<!-- @awaitHelm @test -->
```
tut_retry 4 helm version
```
