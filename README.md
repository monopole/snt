# Kubernetes Cluster Configuration

> __status: under development Oct 2017__

The markdown content in and below the directory
containing this `README` is meant to be a a tutorial
for _kubernetes cluster configuration_.

The tutorial assumes zero kubernetes knowledge.  It
starts with just enough background (more info at
https://kubernetes.io) to let a reader build a cluster
"by hand" from scratch, to motivate the problem of
configuration.  Upon completion, the reader will have
tried two different approaches to cluster
configuration.

### Instructions

Open a terminal and copy/paste code blocks to it,
starting with:

<!-- @makeTutorialWorkingDirectory-->
```
TUT_DIR=$(mktemp -d)
```
All file system use - software installation, source code
and data file creation - will happen in that disposable
directory.

The tutorial can be done directly from the content's
[github repo UX](https://github.com/monopole/snt).  In
any directory, start with `README.md`, then consult
`README_ORDER.txt` to know in which order to visit
remaining directory content.

### For a better experience

Take less than ten seconds (assuming you have
[git](https://git-scm.com/downloads) and
[Go](https://golang.org/doc/install) installed) to
download the content and serve it locally with
[mdrip](https://github.com/monopole/mdrip):

```
cd $TUT_DIR
git clone https://github.com/monopole/snt.git
mkdir -p $TUT_DIR/bin
GOBIN=$TUT_DIR/bin go install github.com/monopole/mdrip
$TUT_DIR/bin/mdrip --mode demo --port 8081 snt
```

Visit http://localhost:8081 to see the material
presented with a navigation menu and one-click copying
of code blocks.

### For an even better experience

Install and run [tmux](https://github.com/tmux/tmux/wiki).

Then, clicking on a code block on a locally served web
page will not only copy the block, it will paste it to
your active tmux session.

Surprisingly handy for demos.
