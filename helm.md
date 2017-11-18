# Helm

> _Configure your cluster using templates._
>
> _Time: 6min_

Before running what follows it's assumed you done
instructions above to start a cluster, i.e. this command
```
kubectl cluster-info
```
provides information on your k8s master.

## Installation

Canonical doc [here](https://github.com/kubernetes/helm/blob/master/docs/install.md).

<!-- @installLatest -->
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

<!-- @initialize -->
```
helm init
```


## Are the lights on?

<!-- @confirmTillerRunning -->
```
kubectl get pods --namespace kube-system | grep tiller
```

```
helm version
```

## Try Canonical Sample App

<!-- @createSampleApp -->
```
cd $TUT_DIR
helm create sample
```

What does the sample contain?
```
find $TUT_DIR/sample
```

### Dig into the details.

Make a helper function:
```
function showSample {
  local f=$TUT_DIR/sample/$1
  clear
  echo "== $f =="
  echo " "
  cat $f
  echo " "
}
```

The metadata about the app goes into `Chart.yaml`:
```
showSample Chart.yaml
```

The `templates` directory holds template files
with 1:1 correspondence to k8s resources types like
Service, Deployment and Ingress.

```
showSample templates/service.yaml
```

```
showSample templates/deployment.yaml
```

```
showSample templates/ingress.yaml
```

Finally, the `values` file is the thing
you want to edit.  The values here are injected by
helm into the templates, which are in turn "rendered"
into k8s objects for instantiation in the cluster.
```
showSample values.yaml
```
