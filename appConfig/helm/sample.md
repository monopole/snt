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

Templates in hand, the `values.yaml` file is the thing
one wants to edit to make instances different.

<!-- @printValues @test -->
```
tut_printSample values.yaml
```

Installation of an app via helm means using these
values to fill in the templates to instantiate
differing k8s resources in the cluster.
