# Instance lifecycle

> _Create instances, then do an upgrade and a rollback._
>
> _Time: 2m_


## Render the instances

Use the templates to create a deployment and service
for the _production_ instance pinned at _v1/release-1_.

Make a couple of tranformation functions:

<!-- @funcRenderR1 @env @test -->
```
function tut_renderRelease1 {
  sed "s/theInst/$1/" |\
  sed "s/theConfigMap/release-1/" |\
  sed "s/:theImgVersion/:1/"
}
```
<!-- @funcRenderR2 @env @test -->
```
function tut_renderRelease2 {
  sed "s/theInst/$1/" |\
  sed "s/theConfigMap/release-2/" |\
  sed "s/:theImgVersion/:2/"
}
```

Apply the resources to the cluster to create the instances:

<!-- @renderProduction @test -->
```
# renderProduction script
pushd $TUT_BASELINE
cat dep-instance.yaml sep.yaml svc-instance.yaml |\
  tut_renderRelease1 production |\
  kubectl apply -f -
popd
```

<!-- @renderStaging @test -->
```
# renderStaging script
pushd $TUT_BASELINE
cat dep-instance.yaml sep.yaml svc-instance.yaml |\
  tut_renderRelease2 staging |\
  kubectl apply -f -
popd
```

The configMaps have nothing that needs replacing - just
apply them:

<!-- @makeServices @test -->
```
kubectl apply -f $TUT_BASELINE/cfg-release-1.yaml
kubectl apply -f $TUT_BASELINE/cfg-release-2.yaml
```

The order in which these resources hit the cluster
doesn't matter; the reconciliation loop will hook
everything up.


### Query the instances

Query the instances, to see the differences in the
response:

<!-- @queryProduction @test -->
```
tut_query tuthello-production mango
```

<!-- @queryStaging @test -->
```
tut_query tuthello-staging papaya
```

> In 'real life', an ingress resource would expose the
> _production_ instance to the world, and hide _staging_,
> letting only internal testing see it.

The `instance:` selector can be used to print
cluster resource associated with a given instance:

<!-- @getStagingPods @test -->
```
kubectl get pods -l instance=staging
```

<!-- @getProductionPods @test -->
```
kubectl get pods -l instance=production
```


## Version Control

The templates, the configMaps, the _render_ functions,
some improved form of the _renderProduction_ and
_renderStaging_ script, etc. should all be checked into
version control (e.g. _git_).

The system should be set up so that when the render
scripts are
[tagged](https://git-scm.com/book/en/v2/Git-Basics-Tagging),
it triggers their application to production.

That way the state of the instances in the cluster
always originates from version control.

## Upgrade production

Assume QA completes _staging_ tests, approving v2 for
rollout along with the new flags and environment vars
in configMap _release-2_.

An upgrade is just another apply.

Modify the _renderProduction_ script and run it:

<!-- @renderProduction @test -->
```
# renderProduction script (modified)
pushd $TUT_BASELINE
cat dep-instance.yaml sep.yaml svc-instance.yaml |\
  tut_renderRelease2 production |\
  kubectl apply -f -
popd
```

<!-- @watchProgress @test -->
```
kubectl rollout status deployment tuthello-production
```

<!-- @query @test -->
```
tut_query tuthello-production mango
```

## Rollback

Post-rollout, users start tweeting about some new bug.

Do one of the following:

### Declarative:  apply previous state

Just go back up and reapply the commands that put
production at _v1/release-1_.

More formally:

* Find the version control change to the _renderProduction_ script.
* Make a reverting change and merge it.
* This change automatically triggers new run of _renderProduction_.

In this way version control remains the source of truth for the cluster state.

### Imperative: rollout undo command

<!-- @imperativeUndo @test -->
```
kubectl rollout undo deployment tuthello-production
```

Confirm it worked

<!-- @query @test -->
```
tut_query tuthello-production mango
```

This command is possible because (at the moment)
deployment retain a history of their configuration.

This is an _imperative command_ - it's incumbent on the
operator to take the time to retroactively edit files
in version control to reflect the state of the system.

### Cleanup

When done, delete everything.

For see the power of labelled instances, do it one
instance at a time:

<!-- @deleteRest @test -->
```
kubectl delete all -l instance=production
kubectl delete all -l instance=staging
kubectl delete --all services
kubectl get all # Shows that everything was marked for deletion.
```

Observations

 * The YAML for the deployment and the service varies
   only in a few places between the instances, and does
   so simply, making it easy to use a template that
   avoided two near identical copies.

 * The configMaps had less in common.  They set the
   flag and environment variables that make the
   instances different.

 * The configMap cannot help with non-container
   aspects of app instance configuration like resource
   names, labels and port specifications.
