# Immutable Maps

> _Immutable objects are the best objects._
>
> _Time: 3m_

A change to a deployment specification triggers a k8s
reconciler, whose job it is to change the cluster to
agree with the deployment spec.

As the previous section makes clear, this is not the
case for configMaps.

Instead of changing a map, create a new map, and modify
the deployment to point to the new map instead of the
old.

Create a second map called `cfg-cilantro`:

<!-- @cilantroMap @test -->
```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: cfg-cilantro
data:
  altGreeting: "Irasshaimase"
  enableRisky: "false"
EOF
```

<!-- @descConfigMap @test -->
```
kubectl describe configmap cfg-cilantro
```

Confirm existing behavior:
<!-- @queryIt @test -->
```
tut_query svc-eggplant lime
```

Write a function to change the configMap:
<!-- @funcSwapAndApply @test -->
```
function tut_swapAndApply {
  cat $TUT_DIR/raw/dep-kale.yaml |\
    sed "s/$1/$2/" |\
    kubectl apply -f -
}
```


Apply the change:

<!-- @changeToConfig2 @test -->
```
tut_swapAndApply cfg-parsley cfg-cilantro
```

Query again, until the change appears:

<!-- @queryIt @test -->
```
tut_query svc-eggplant lime
```

Change it back and forth, and rapidly query to watch the turnover.

<!-- @changeToC1WithQuery @test -->
```
tut_swapAndApply cfg-cilantro cfg-parsley
for i in {1..10}; do
  tut_query svc-eggplant toParsley
  sleep 0.3
done
```

<!-- @changeToC2WithQuery @test -->
```
tut_swapAndApply cfg-parsley cfg-cilantro
for i in {1..10}; do
  tut_query svc-eggplant toCilantro
  sleep 0.3
done
```

That's it.

<!-- @cleanup @test -->
```
kubectl delete deployment dep-kale
kubectl delete configmap cfg-parsley
kubectl delete configmap cfg-cilantro
```
