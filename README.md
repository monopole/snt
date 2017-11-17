# Kubernetes Cluster Configuration

> _A tutorial on installing customized cluster-level
> "apps"._

> __status: under development Oct 2017__

The content in the directory containing this `README`
introduces the reader to aggregate k8s cluster
configuration and management.

No k8s knowledge or existing infrastructure is assumed.
Instructions to start a cluster are included to allow
self-contained test coverage for the tutorial.
Nevertheless, background discussion is kept to a
minimum.

The tutorial introduces pods and deployments in the
course of building a cluster from scratch, to motivate
the problem of configuration.  The tutorial finishes by
working with two cluster configuration tools.

See https://kubernetes.io for more documentation and
tutorials.

## Using the tutorial

Open a bash shell and copy/paste code blocks to it,
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
[github repo UX](https://github.com/monopole/snt)
(i.e. what you're likely reading right now).  Start
with the `README.md` in any directory, then consult
`README_ORDER.txt` to see the order in which to visit
remaining content.

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

Then visit http://localhost:8081 to see the material
presented with a navigation menu and one-click copying
of code blocks and check marks to show completion.

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
