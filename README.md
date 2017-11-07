# Kubernetes Cluster Configuration

> __status: under development Oct 2017__

The markdown content in and below the directory
containing this `README` is meant to be a a tutorial
for _kubernetes cluster configuration_.

The tutorial assumes zero kubernetes knowledge.  It
provides just enough background to let a reader build a
cluster "by hand" from scratch, to motivate the problem
of configuration.  The canonical source of kubernetes
learning is https://kubernetes.io.

The tutorial concludes by working through two
different approaches to cluster configuration.

### Instructions

Open a terminal and copy/paste command blocks to it as
you read, starting with the following block:

<!-- @makeTutorialWorkingDirectory-->
```
TUT_DIR=$(mktemp -d)
```
All file writing - software installation, source code
and data file creation - will happen in that disposable
directory.

The tutorial can be done directly from the content's
[github repo UX](https://github.com/monopole/snt).  In
any directory, start with the `README`, then read the
`SORT_MD.txt` file to know in which order to read the
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
mdrip --mode demo --port 8081 snt
```

Visit http://localhost:8081 to see the material
presented with a properly ordered navigation menu and
one-click copying of command blocks.

### For an even better experience

Install and run [tmux](https://github.com/tmux/tmux/wiki).

Then, clicking on a command block on a locally served
webpage will both copy and paste the block to your
active tmux session.

This is surprisingly handy for demos.
