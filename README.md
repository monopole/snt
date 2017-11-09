# Kubernetes Cluster Configuration

> _A tutorial on installing customized cluster-level
> "apps"._

> __status: under development Oct 2017__

The directory containing this `README` holds a brief
introduction to aggregate k8s cluster configuration and
management.

The tutorial introduces pods and deployments in the
course of building a cluster from scratch, to motivate
the problem of configuration.  The tutorial finishes by
working with two cluster configuration tools.

No k8s knowledge assumed, but background discussion is
kept to a minimum.  See https://kubernetes.io for more
documentation and tutorials.

### Instructions

Open a terminal and copy/paste code blocks to it,
starting with:

<!-- @makeTutorialWorkingDirectory-->
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

The tutorial can be done directly from the content's
[github repo UX](https://github.com/monopole/snt).
Start with the `README.md` in any directory, then
consult `README_ORDER.txt` to see the order in which to
visit remaining content.

### For a better experience

Take less than ten seconds (assuming you have
[git](https://git-scm.com/downloads) and
[Go](https://golang.org/doc/install) installed) to
download the content and serve it locally with
[mdrip](https://github.com/monopole/mdrip):

```
git clone https://github.com/monopole/snt.git $TUT_DIR/snt
mkdir -p $TUT_DIR/bin
GOBIN=$TUT_DIR/bin go install github.com/monopole/mdrip
$TUT_DIR/bin/mdrip --mode demo --port 8081 $TUT_DIR/snt
```

Visit http://localhost:8081 to see the material
presented with a navigation menu and one-click copying
of code blocks.

### For an even better experience

Install and run [tmux](https://github.com/tmux/tmux/wiki).

Then, clicking on a code block on a locally served web
page will immediately paste the block to your active
tmux session.  Handy for demos.
