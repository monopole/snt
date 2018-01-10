# Eschew Changing ConfigMaps

> _Optional section: Change a configmap to see why its not a good thing._
>
> _Time: 1m_

Apply a change to `cfg-french` that changes the greeting:

<!-- @applyMapChange @test -->
```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: cfg-french
data:
  altGreeting: "Bienvenue"
  enableRisky: "false"
EOF
```

<!-- @descConfigMap @test -->
```
kubectl describe configmap cfg-french
```

A query shows that `tuthello` is still using
_Bonjour_:

<!-- @curlService @test -->
```
tut_query svc-eggplant lemon | grep Bonjour
```

### Mixed states

Define function to delete a random pod:

<!-- @funcDelRandomPod @env @test -->
```
function tut_deleteRandomPod {
  local tmpl=`cat <<EOF
{{range .items -}}
{{.metadata.name}}
{{end}}
EOF
`
  local victim=$(\
      kubectl get -o go-template="$tmpl" pods |\
      sort -R | head -n 1)
  kubectl delete pod $victim
}
```

Delete one pod at a time and watch as new configuration
is adopted by new pods.

<!-- @deleteOnePod @test -->
```
tut_deleteRandomPod
sleep 2
```

Repeat this query enough times to hit all pods
shows emit both a _Bonjour_ and a _Bienvenue_:

<!-- @tryQuery  -->
```
tut_query svc-eggplant orange
```

The cluster can be placed in an undesirable mixed state
by changing the config and deleting a single pod.

Get everything in sync again by deleting all the pods
to force recreation at the latest config:

<!-- @deleteAllPods @test -->
```
kubectl delete --all pods
```

Confirm the new greeting _Bienvenue_ is always served:
<!-- @tryQuery @test -->
```
tut_query svc-eggplant orange
```

The point is of the section is to recall that pods can
come and go at any time, and there’s no way to know
which version of the map was in force when the pod was
created.
