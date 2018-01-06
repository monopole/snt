# The Sample App

> _Explore the sample app built into helm._
>
> _Time: 2m_


Helm has a `create` command that makes a directory
populated with the files necessary to make a helm app.
The argument to `create` is the name given to the new
app.

Call it _potato_:

<!-- @makeSample @test -->
```
helm create $TUT_DIR/potato
```

<!-- @listFiles @test -->
```
find $TUT_DIR/potato
```

## The Chart

Self-explanatory metadata about the app goes into a
single manifest file called `Chart.yaml`.

The argument to `create` appears in this file as
well.


<!-- @showChart @test -->
```
cat $TUT_DIR/potato/Chart.yaml
```

## Templates

[Go language template]: https://golang.org/pkg/text/template/

The manifest is associated with a
`templates` directory holding [Go language template]
files that correspond to k8s resources types like
_service_ and _deployment_.

The templates hold the commonality instances will share.

<!-- @showService @test -->
```
clear
cat $TUT_DIR/potato/templates/service.yaml
```

<!-- @showDeployment @test -->
```
clear
cat $TUT_DIR/potato/templates/deployment.yaml
```

## Values

The `values.yaml` holds information to
make instances different.

<!-- @showValues @test -->
```
clear
cat $TUT_DIR/potato/values.yaml
```

Installation of an app via helm means using these
values to fill in the templates to instantiate
k8s resources in the cluster.
