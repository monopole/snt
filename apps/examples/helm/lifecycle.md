# tuthello Lifecycle

> _Install and run the instances of tuthello via helm._
>
> _Time: 2m_

Check the files created in the previous step with
helm's linter:

<!-- @lintApp @test -->
```
helm lint $TUT_DIR/tuthello
```

The linter doesn't actually render the templates, so
won't find Go template errors.

Do a dry run to force a rendering, which will fail on
template errors:

<!-- @examineProduction @test -->
```
helm install \
    --dry-run --debug \
    --name production \
    -f $TUT_DIR/tuthello/release-1.yaml \
    $TUT_DIR/tuthello
```

If the above fails, fix your Go templates.

## Run the instances

Install the production instance:

<!-- @installProduction @test -->
```
helm install \
    --name production \
    -f $TUT_DIR/tuthello/release-1.yaml \
    $TUT_DIR/tuthello
```

Install the staging instance:
<!-- @installStaging @test -->
```
helm install \
    --name staging \
    -f $TUT_DIR/tuthello/release-2.yaml \
    $TUT_DIR/tuthello
```

[before]: review/configuration/lifecycle

Confirm the existence of the two instances
by doing the same queries used [before]:

<!-- @queryInstances1 @test -->
```
tut_query tuthello-production mango
tut_query tuthello-staging papaya
```
## Upgrade and rollback

[before]: review/configuration/lifecycle

As [before], assume QA certifies the release-2
configuration on the staging instance, meaning
it's time to push release-2 to production:

<!-- @helmUpgrade @test -->
```
helm upgrade production \
    -f $TUT_DIR/tuthello/release-2.yaml \
    $TUT_DIR/tuthello
```

Confirm that the behavior of the instances is identical:

<!-- @queryInstances2 @test -->
```
tut_query tuthello-production mango
tut_query tuthello-staging papaya
```

Again assume our unhappy user found a bug, and we have
to do a rollback:

<!-- @helmRollback @test -->
```
helm rollback production 1
```

Did the rollback work?  Try this a few times:

<!-- @queryInstances3 @test -->
```
tut_query tuthello-production mango
```

Helm's _tiller_ retains information about these
changes.

What the operator does with source control to track
these changes is up to them.

That all; clean up:

<!-- @cleanup @test -->
```
helm delete --purge staging || true
helm delete --purge production || true
```
