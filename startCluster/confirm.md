# Can kubectl Talk to your Cluster?

> _Are the lights on?_
>
> _Time: 10sec_


Regardless of which cluster you've set up,
you should now be able to talk to it via kubectl.

Confirm you have at least one node...

<!-- @getNodes @test @debug -->
```
kubectl get nodes
```

...but no pods.
<!-- @getPods -->
```
kubectl get pods
```

Get node details, noting the cpu and memory numbers in the capacity section.


<!-- @getPods @test -->
```
kubectl describe nodes
```

Lastly, define a function used in the remaining sections.

<!-- @funcIsMiniKube @test @debug -->
```
function tut_isMinikube() {
  local tmpl='{{ with index .items 0}}{{.metadata.name}}{{end}}'
  local firstNodeName=$(kubectl get -o go-template="$tmpl" nodes)
  [[ "$firstNodeName" == "minikube" ]]
}
if tut_isMinikube; then
  echo "Using minikube"
else
  echo "Using GKE"
fi
```

