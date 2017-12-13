# Kubernetes Cluster Configuration

_A tutorial on installing customized cluster-level "apps"._

> __status: under development Oct 2017__

This tutorial requires no kubernetes knowledge.
It focusses on command execution,
eschewing deeper discussion that can be found at
https://kubernetes.io and many other sites.

To establish context for the main topic, this
tutorial starts with a walkthrough of manual
configuration (individual commands to create a pod,
create a service, etc.).

Context established, it compares different approaches
to app-level configuration.

### Requirements

[Go](https://golang.org/doc/install),
[git](https://git-scm.com/downloads) and bash.
Some command blocks use bash-specific syntax.

Further requirements arise from the choice of where you
run your cluster (discussed next).

### Usage

Begin by creating a disposable working directory:

```
TUT_DIR=$(mktemp -d)
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
