# Helm's Canonical Sample App

Make the sample app:

<!-- @makeSampleApp -->
```
cd $TUT_DIR
helm create sample
```

What does the sample contain?
```
find $TUT_DIR/sample
```

Make a helper function for use below:

<!-- @funcPrintSample @env -->
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

## Metadata

Self-explanatory
metadata about the app goes into `Chart.yaml`:

```
tut_printSample Chart.yaml
```


## Templates


The `templates` directory holds template files
with 1:1 correspondence to k8s resources types like
_Service_, _Deployment_ and _Ingress_.

```
tut_printSample templates/service.yaml
```

```
tut_printSample templates/deployment.yaml
```

```
tut_printSample templates/ingress.yaml
```

## Values

Templates in hand, the `values.yaml` file is the thing
one wants to edit.

The values are injected by helm into the templates,
which are in turn "rendered" into k8s objects for
instantiation in the cluster.

```
tut_printSample values.yaml
```
