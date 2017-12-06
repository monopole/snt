# Kubernetes Cluster Configuration

> _A tutorial on installing customized cluster-level
> "apps"._

> __status: under development Oct 2017__

This tutorial describes configuring a cluster managed
by kubernetes (aka _k8s_).

No k8s knowledge or existing infrastructure is assumed,
but discussion is kept to a minimum to focuss on
practice with real commands.  See https://kubernetes.io
for more information.

The bulk of the tutorial describes "manual"
configuration using kubectl and the raw k8s api.  It
concludes by using two very different configuration
tools to better automate the preceding process.

## Using the tutorial

Open a bash shell and copy/paste code blocks
to it, starting with:

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

If you are viewing this content in its raw form on
[github repo UX](https://github.com/monopole/snt), you
can start with the `README.md` in any directory, then
consult `README_ORDER.txt` to see the order in which to
visit remaining content.

### For a better experience

Take less than ten seconds (assuming you have
[git](https://git-scm.com/downloads) and
[Go](https://golang.org/doc/install) installed) to
serve the content locally with
[mdrip](https://github.com/monopole/mdrip):

```
mkdir -p $TUT_DIR/bin
GOBIN=$TUT_DIR/bin go install github.com/monopole/mdrip
$TUT_DIR/bin/mdrip --mode demo --port 8001 gh:monopole/snt
```

Then visit http://localhost:8001 to see the material
presented with a navigation menu, progress checks,
and and one-click copying of code blocks.

### For an even better experience

Install and run [tmux](https://github.com/tmux/tmux/wiki),
e.g.

```
sudo apt-get install tmux
tmux
```

Then, clicking on a code block on a locally served web
page will immediately paste the block to your active
tmux session.  Handy for demos.
