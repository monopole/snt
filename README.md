# Kubernetes Cluster Configuration

_A tutorial on installing customized cluster-level apps._

> __status: under development Nov 2017__

This tutorial requires no knowledge of kubernetes,
also known as _k8s_.

It focusses on command execution, eschewing deeper
discussion that can be found at https://kubernetes.io
and many other sites.

To establish context for the main topic, this tutorial
starts with a walk-through of manual configuration
(individual commands to create a pod, create a service,
etc.).  Context established, it compares different
approaches to app-level configuration.

### Requirements

[Go](https://golang.org/doc/install),
[git](https://git-scm.com/downloads)
and [curl](https://github.com/curl/curl).

The bash shell is implicitly required, as the
command blocks use bash syntax.

Further requirements arise from the choice of where you
run your cluster (discussed next).

### Usage

Begin by creating a disposable working directory:

<!-- @defineEnv @test @debug -->
```
# Use fixed (rather than random) directory
# to ease debugging command blocks.
# TUT_DIR=$(mktemp -d)
TUT_DIR=$TMPDIR/k8s_config_tutorial
```

<!-- @resetTmpDir @test -->
```
if [ -d "$TUT_DIR" ]; then
  /bin/rm -rf $TUT_DIR
fi
mkdir $TUT_DIR
```

With the exception of the optional gcloud installation
(discussed later), all file system use happens in this
disposable directory.

```
echo $TUT_DIR; ls $TUT_DIR
```

Cleanup is just

> ```
> rm -rf $TUT_DIR
> ```

which your OS will eventually do on its own.

