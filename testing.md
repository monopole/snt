# Tutorial Testing

> _How to test or rerun all or part of the tutorial._
>
> _Time: ~5min_

## Prerequisites

[mdrip]: https://github.com/monopole/mdrip

[mdrip] rips command blocks from markdown for execution.

<!-- @installMdrip -->
```
GOBIN=$TMPDIR go install github.com/monopole/mdrip
alias mdrip=$TMPDIR/mdrip
```


The tutorial content lives on github.
It's convenient for what follows to explicitly
download it, rather than have mdrip do so with
every invocation:

<!-- @installContent -->
```
TUT_REPO=https://github.com/monopole/snt
/bin/rm -rf $TMPDIR/tutorial
git clone $TUT_REPO $TMPDIR/tutorial
cd $TMPDIR/tutorial
```


## Test the entire tutorial

<!-- @testAllContent -->
```
mdrip --mode test --label test --blockTimeOut 15m .
echo $?
```

A zero from __`echo $?`__ means
all blocks with the `@test` label ran without error. The
blockTimeout allows downloads of VM ISO files over
slow wifi, the slowest step in the test.

A failure reports the command block that failed,
along with the failing block's `stdout` and `stderr`.

To see what the test runs, extract the blocks to stdout:

<!-- @printScript -->
```
mdrip --label test . | more
```

## Re-establish lost environment

Suppose you unintentionally close your working shell,
after doing all the downloads and starting a cluster.

To quickly re-establish the environment, run only blocks
labelled `@env`:

```
eval "$(mdrip --label env .)"
```

Obviously this depends on the placement of `@env` labels
in the markdown.


## Fast forward

One can use mdrip's code extraction to run through
the tutorial quickly up to a certain point.

`mdrip` accepts file names as arguments, and runs them
in the order specified.

To emulate mdrip's behavior when given a directory tree
as an argument, specify `README.md` first, followed by
file names as they appear in a directory's
`README_ORDER.txt` file, recursively descending as
needed, stopping where desired.

E.g. to go from zero to the point where one has working
minikube cluster with a pod, try:

```
eval "$(mdrip --label test \
    ./README.md \
    ./environment.md \
    ./hosting/minikube.md \
    ./hosting/confirm.md \
    ./tuthello.md \
    ./containerize.md \
    ./review/namespaces.md \
    ./review/pods.md \
)"
```

This runs in your current shell, modifying it, unlike
`--mode test`, which runs everything in a subprocess.

Also, unlike `--mode test`, the above command won't
fail on error (it just barges on as if you were typing
commands), it won't attempt to capture and report
output from failing blocks, and a success (`$?==0`) at
the end only means the last command succeeded, not the
entire thing.


## Early-stop testing

Just do the above, in `--mode test`, without the `eval`:

```
mdrip --mode test --label test --blockTimeOut 15m \
    ./README.md \
    ./environment.md \
    ./hosting/minikube.md \
    ./hosting/confirm.md \
    ./tuthello.md \
    ./containerize.md \
    ./review/namespaces.md \
    ./review/pods.md  # stop whereever
```

## Late-start testing

When _debugging blocks in test mode_, it's sometimes
desirable to run some part of the tutorial in test mode
without rerunning everything from the start.  But test
mode is a subprocess. If not re-run from the start, its
environment won't be established.

The workaround is to re-establish the environment as
described above (all global variables defined therein
should be preceded by `export`), then export all the
bash functions as a next step:

```
while IFS= read -r line; do export -f $line; done < <(
    declare -F | grep tut_ | awk '{ print $3 }')
```

Then run only the file you want to test, e.g.:

```
mdrip --mode test --label test review/services.md
```

For more logging, add

> ```
> --alsologtostderr --v 2 --stderrthreshold INFO
> ```
