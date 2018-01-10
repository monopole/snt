# Instances of the Sample

> _Install two instances of the potato app._
>
> _Time: 2m_

Make two instances of potato, called
_staging_ and _production_.

Launch an instance of potato, naming it _staging_:

<!-- @installStaging @test -->
```
helm install $TUT_DIR/potato --name staging
```

As noted earlier, the `potato/templates` directory has
(at least) the files

> ```
> deployment.yaml
> service.yaml
> ```

The install rendered these templates into their
respective k8s service and deployment resources, and
given them both the same name, _staging-potato_:

<!-- @getStaging @test -->
```
kubectl get service staging-potato
kubectl get deployment staging-potato
```

Install one more instance, calling it _production_:

<!-- @installProduction @test -->
```
helm install $TUT_DIR/potato --name production
```

Print the difference between the two service instances:

<!-- @diffService  -->
```
diff \
  <(kubectl describe service production-potato) \
  <(kubectl describe service staging-potato)
```

This shows that in addition to prefixing the resource
names with the instance name (`production-`,
`staging-`), helm has added the selectors:

> ```
> app=potato,release=production
> app=potato,release=staging
> ```

helm uses the word _release_ for what this tutorial
calls an _instance_, and adds that as a selector to
each resource, usable thus:

<!-- @getAllStaging @test -->
```
kubectl get all -l release=staging
```

Helm has its own notion of state, maintained by tiller:

<!-- @list @test -->
```
helm list
```

Delete the instances:

<!-- @deleteInstances @test -->
```
helm delete staging
helm delete production
```

Use kubectl get again to confirm that the
helm-recreated resources are gone.

Tiller is still running, and it remembers the instances
after deletion.

<!-- @listDeleted @test -->
```
helm list --deleted
```

Tiller can report the rendered templates (the k8s resources)
associated with an instance:

<!-- @dumpYaml @test -->
```
helm get manifest staging
```

Do some cleanup:
<!-- @cleanup @test -->
```
kubectl delete deployments --all
kubectl delete services --all
kubectl delete configmaps --all
helm delete --purge staging || true
helm delete --purge production || true
```
