# Kubernetes Cluster Configuration

> _A tutorial on installing customized cluster-level
> "apps"._

> __status: under development Oct 2017__

This tutorial requires no kubernetes knowledge.
To establish context for the closing topics, this
tutorial starts with a walkthrough of _manual_
configuration via kubectl.

The tutorial concludes by comparing different
approaches to _app-level_ configuration.

The tutorial focusses on command execution.  It
eschews deeper discussion that can be found at
https://kubernetes.io and many other sites.

### Requirements

[Go](https://golang.org/doc/install),
[git](https://git-scm.com/downloads) and bash.
Some command blocks use bash-specific syntax.

## Usage

Open a `bash` shell and copy/paste code blocks to
it, starting with:

<!-- @makeWorkingDir -->
```
TUT_DIR=$(mktemp -d)
```

With the exception of the optional gcloud installation
(discussed later), all file system use
happens in this disposable directory.

Cleanup up is just

> ```
> rm -rf $TUT_DIR
> ```

## If you are reading this on github...

If you are viewing this content in its raw form on
[github](https://github.com/monopole/snt),
start with the `README.md` in any directory, then
consult `README_ORDER.txt` to see the order in which to
visit remaining content in the given directory.

### For a better experience

Take less than ten seconds to serve the content locally with
[mdrip](https://github.com/monopole/mdrip):

```
mkdir -p $TUT_DIR/bin
GOBIN=$TUT_DIR/bin go install github.com/monopole/mdrip
$TUT_DIR/bin/mdrip --mode demo --port 8001 gh:monopole/snt
```

Then visit http://localhost:8001 to see the material
presented with a navigation menu, progress checks,
and one-click copying of code blocks.

### For an even better experience

Additionally install and run [tmux](https://github.com/tmux/tmux/wiki),
e.g.

```
sudo apt-get install tmux
tmux
```

Then, clicking on a code block on a locally served web
page will immediately paste the block to your active
tmux session.  Handy for demos (no need to retarget the
mouse).
