# Helm's Canonical Sample App

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