# Indentity transformation

> _Kinflate is a resource transformer._
>
> _Time: 30s_

Kinflate's job is to read a manifest and resources and
generate _transformed_ (customized) resources that can
be directly applied to a cluster.

For later comparisons, _save the current
uncustomized output_ (a YAML stream)
before doing any customizations.

<!-- @noCustomization @test -->
```
kinflate inflate -f $TUT_DAM >$TUT_TMP/original_out
```

kinflate expects to find `Kube-manifest.yaml` in the
_-f_ directory.  The manifest tells kinflate which
resources to use.

As the manifest and its containing directory now stand,
this command spits out _unmodified resources_.  The
output could be piped directly to kubectl:

> ```
> kinflate inflate -f $TUT_DAM | kubectl apply -f -
> ```

The resulting change to the cluster would be no
different than using kubectl directly,

> ```
> kubectl apply -f $TUT_DAM
> ```

as done [before](apps/examples/DAM/base),
because we've not done any customization yet.
