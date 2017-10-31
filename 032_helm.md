# Helm

## Installation

Canonical doc [here](https://github.com/kubernetes/helm/blob/master/docs/install.md).

<!-- @getLatest -->
```
apis=https://storage.googleapis.com
curl -Lo $TUT_DIR/helm.tgz $apis/kubernetes-helm/helm-v2.7.0-linux-amd64.tar.gz
```

<!-- @installLatest -->
```
cd $TUT_DIR
gunzip <helm.tgz | tar xvf -
rm helm.tgz
mv linux-amd64/helm .
/bin/rm -rf linux-amd64
```

```
$TUT_DIR/helm version
```

<!-- @initialize -->
```
$TUT_DIR/helm init
```


## Basics

<!-- @initialize -->
```
$TUT_DIR/helm ls
```
