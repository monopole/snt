# First order customization

> _What's the simplest thing to do?_
>
> _Time: 1m_

The bulk of the fields in the manifest exist
to support app instance customization.

## First run

Kinflate's job is to read a manifest and resources and
generate customized resources that can be directly
applied to a cluster.

One can do so immediately:

<!-- @runKinflate @test -->
```
kinflate inflate -f $TUT_DAM/manifest
```

kinflate expects to find `Kube-manifest.yaml`
in this directory, and uses it to find resources.

As the manifest now stands, this command spits out
unmodified YAML containing all the resources.  The
outout could be piped directly to kubectl:

> ```
> kinflate inflate -f $TUT_DAM/manifest |\
>     kubectl apply -f -
> ```

The result is  no different than using kubectl directly,

> ```
> kubectl apply -f $TUT_DAM/manifest
> ```

as done [before](apps/examples/DAM/base).

### First order customization

Before doing any customization, save the current
uncustomized output:

<!-- @noCustomization @test -->
```
kinflate inflate -f $TUT_DAM/manifest >$TUT_TMP/original
```

First order customization in DAM is done by modifying
the `Kube-manifest.yaml` directly.

The simplest form of customization is to modify names.
In DAM, this is done by adding a prefix.

Be sure the manifest is in original form:
<!-- @resetManifest @test -->
```
cp $TUT_ORG_MANIFEST $TUT_DAM_MANIFEST
```

Add one line to the manifest:
<!-- @namePrefix @test -->
```
echo "namePrefix: acme-" >> $TUT_DAM_MANIFEST
```

Run `kinflate`:

<!-- @runKinflate @test -->
```
kinflate inflate -f $TUT_DAM/manifest >$TUT_TMP/customized
```

Compare the differences to see the effect:

<!-- @runKinflate @test -->
```
diff $TUT_TMP/original $TUT_TMP/customized
```

At this point, an end user could check the manifest and
its resources into local version control, and have a
cluster app that would create resources with names that
differed from off-the-shelf names by a prefix.
