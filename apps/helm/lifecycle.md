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
    -f $TUT_DIR/tuthello/production.yaml \
    $TUT_DIR/tuthello
```

If the above fails, fix your templates.

## Run the instances

Install the production instance:

<!-- @installProduction @test -->
```
helm install \
    --name production \
    -f $TUT_DIR/tuthello/production.yaml \
    $TUT_DIR/tuthello
```

Install the staging instance:
<!-- @installStaging @test -->
```
helm install \
    --name staging \
    -f $TUT_DIR/tuthello/staging.yaml \
    $TUT_DIR/tuthello
```

Query them:

<!-- @queryInstances @test -->
```
tut_query tuthello-production mango
tut_query tuthello-staging papaya
```

These queries are the same as those done in
[/review/configuration/instances]([/review/configuration/instances].)
