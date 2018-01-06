# Instance lifecyle

> _Create instances, perform an upgrade, and
> delete them._
>
> _Time: 2m_


## Render the instances

Use the templates to create a deployment and service
for the _production_ instance pinned at v1/release-1.

Make a couple of tranformation functions:

<!-- @funcRenderR1 @env @test -->
```
function tut_renderRelease1 {
  sed "s/myInst/$1/" |\
  sed "s/cfgmap/release-1/" |\
  sed "s/:imgVersion/:1/"
}
```
<!-- @funcRenderR2 @env @test -->
```
function tut_renderRelease2 {
  sed "s/myInst/$1/" |\
  sed "s/cfgmap/release-2/" |\
  sed "s/:imgVersion/:2/"
}
```

Apply the resources to the cluster to create the instances:

<!-- @renderProduction @test -->
```
# renderProduction script
pushd $TUT_DIR/raw
cat dep-instance.yaml sep.yaml svc-instance.yaml |\
  tut_renderRelease1 production |\
  kubectl apply -f -
popd
```

<!-- @renderStaging @test -->
```
# renderStaging script
pushd $TUT_DIR/raw
cat dep-instance.yaml sep.yaml svc-instance.yaml |\
  tut_renderRelease2 staging |\
  kubectl apply -f -
popd
```

The configMaps have nothing that needs replacing - just
apply them:

<!-- @makeServices @test -->
```
kubectl apply -f $TUT_DIR/raw/cfg-release-1.yaml
kubectl apply -f $TUT_DIR/raw/cfg-release-2.yaml
```

The order in which these resources hit the cluster
doesn't matter; the reconciliation loop will hook
everything up.


### Query the instances

Query the instances, to see the differences in the
response:

<!-- @query @test -->
```
tut_query tuthello-production mango
tut_query tuthello-staging papaya
```

> In 'real life', an ingress resource would expose the
> _production_ instance to the world, and hide _staging_,
> only letting internal testing see it.

The `instance:` selector can be used to print
cluster resource associated with a given instance:

<!-- @getSelector @test -->
```
kubectl get pods -l instance=staging
```
## Source Control

The templates, the configMaps, the _render_ functions,
some improved form of the _renderProduction_ and
_renderStaging_ script, etc. should all be checked into
source control.

The system should be set up so that any change to the
these render scripts would trigger its execution.

That way the state of the instances in the cluster
always originates from source control.


## Upgrade production

Assume QA completes _staging_ tests, approving v2 for
rollout along with the new flags and environment vars
in configMap release-2.

An upgrade is just another apply.
Modify the _renderProduction_ script and run it:

<!-- @modifyProduction @test -->
```
# renderProduction script (modified)
pushd $TUT_DIR/raw
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

Do one or the other of the following:

### Overwrite the apply

Just go back up and reapply the commands that put
production at v1/release-1.

Ideally this is a `git revert` to the
_modifyProduction_ script in source control, that
automagically triggers script execution.

### Break-glass emergency imperative

<!-- @emergencyUndo @test -->
```
kubectl rollout undo deployment tuthello-production
```

Confirm it worked

<!-- @query @test -->
```
tut_query tuthello-production mango
```

This is possible because deployments retain a history.

This is an imperative command - it's up to the operator
to edit files in source control to reflect the state of the system.

If the edits are done correctly, any automatic application
of the changes will have zero effect (harmlessly confirming
the change is correct).  Of course, it would be wise to
use `kubectl diff` first.

### Cleanup

When done, delete everything.

For entertainment, do it one instance at a time:

<!-- @deleteRest @test -->
```
kubectl delete all -l instance=production
kubectl delete all -l instance=staging
kubectl delete --all services
kubectl get all
```

Observations

 * The YAML for the deployment and the service varies
   only in a few places between the instances, and does
   so simply, making it easy to use a template that
   avoided two near identical copies.

 * The config maps had less in common, since that's
   where the flag and environment variables are set
   that makes the instances different.

 * The config map cannot help with non-container
   aspects of app instance configuration like resource
   names, labels and port specifications.

Much more on app instances in the next section.
