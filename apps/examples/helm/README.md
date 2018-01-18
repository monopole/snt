# Helm

> _Configure your cluster using Go templates._
>
> _Time: 8m_

[Helm](https://helm.sh/) manages app instances via template substitution.

## Install to disk

[install doc]: https://github.com/kubernetes/helm/blob/master/docs/install.md

Try the following (or visit the canonical [install doc]):

<!-- @installHelm @test -->
```
function tut_installHelm {
  local tarball=$1
  local apis=https://storage.googleapis.com
  pushd $TUT_TMP
  curl --fail --location --silent \
      -o helm.tgz $apis/kubernetes-helm/$tarball
  gunzip <helm.tgz | tar xvf -
  rm helm.tgz
  mv linux-amd64/helm $TUT_BIN
  /bin/rm -rf linux-amd64
  popd
}
tut_installHelm helm-v2.7.0-linux-amd64.tar.gz
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

Install tiller, and create whatever other state helm needs:

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
