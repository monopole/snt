# The Sample App

Make the bundle of files associated with helm's
canonical sample app:

<!-- @makeTheSample @test -->
```
cd $TUT_DIR
helm create sample
```

<!-- @listContents @test -->
```
find $TUT_DIR/sample
```

Make a helper function for use below:

<!-- @funcPrintSample @env @test -->
```
function tut_printSample {
  local f=$TUT_DIR/sample/$1
  clear
  echo "== $f =="
  echo " "
  cat $f
  echo " "
}
```

## The Chart

Self-explanatory metadata about the app goes into a
single manifest file called `Chart.yaml`:

<!-- @printChart @test -->
```
tut_printSample Chart.yaml
```

## Templates

[Go language template]: https://golang.org/pkg/text/template/

The manifest (aka chart) is associated with a
`templates` directory holding [Go language template]
files that correspond to k8s resources types like
_service_ and _deployment_.

The templates hold the commonality instances will share.

<!-- @printService @test -->
```
tut_printSample templates/service.yaml
```

<!-- @printDeployment @test -->
```
tut_printSample templates/deployment.yaml
```

## Values

The `values.yaml` holds information to
make instances different.

<!-- @printValues @test -->
```
tut_printSample values.yaml
```

Installation of an app via helm means using these
values to fill in the templates to instantiate
differing k8s resources in the cluster.

## Try the sample

Install and remove two instances
<!-- @noteCurrentState @test -->
```
kubectl get services
kubectl get deployments
```

<!-- @installOne @test -->
```
helm install $TUT_DIR/sample --name sample1
kubectl get services
kubectl get deployments
```

<!-- @installAnother @test -->
```
helm install $TUT_DIR/sample --name sample2
kubectl get services
kubectl get deployments
```

<!-- @list @test -->
```
helm ls
```

<!-- @delete @test -->
```
helm delete sample1
helm delete sample2
helm ls
kubectl get services
kubectl get deployments
```
