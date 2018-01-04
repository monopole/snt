# Can kubectl Talk to your Cluster?

> _Are the lights on?_
>
> _Time: 10sec_


Regardless of which cluster you've set up,
you should now be able to talk to it via kubectl.

Confirm you have at least one node...

<!-- @getNodes @test -->
```
kubectl get nodes
```

...but no pods.
<!-- @getPods -->
```
kubectl get pods
```

Get node details, noting the cpu and memory numbers in
the capacity section.


<!-- @getPods @test -->
```
kubectl describe nodes
```

Lastly, define a function used in the remaining sections.

<!-- @funcIsMiniKube @env @test -->
```
function tut_isMinikube {
  local t='{{ with index .items 0}}{{.metadata.name}}{{end}}'
  local name=$(kubectl get -o go-template="$t" nodes)
  [[ "$name" == "minikube" ]]
}
if tut_isMinikube; then
  echo "Using minikube"
else
  echo "Presumably using GKE"
fi
```
