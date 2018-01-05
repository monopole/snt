# Tutorial Testing

> _How to test or rerun all or part of the tutorial._
>
> _Time: ~5min_

## Prerequisites

[mdrip]: https://github.com/monopole/mdrip
[content]: https://github.com/monopole/snt

Download the tutorial [content] and the [mdrip] tool:

<!-- @download -->
```
TST_DIR=$(mktemp -d)
repo=https://github.com/monopole/snt.git
git clone $repo $TST_DIR/tutorial
GOBIN=$TST_DIR go install github.com/monopole/mdrip
```

All commands below assume

```
PATH=$TST_DIR:$PATH
cd $TST_DIR/tutorial
```

## Test the entire tutorial

The following is meant to be used as a CI/CD step:

<!-- @testAllContent -->
```
mdrip --mode test --label test --blockTimeOut 15m .
echo $?
```
No output (other than zero from __`echo $?`__)
means all blocks with the `@test` label ran
without error.

A failure reports the command block that failed,
along with the failing block's `stdout` and `stderr`.

> _The long blockTimeout allows downloads
> of VM ISO files over slow wifi,
> the slowest step in the test._

## Fast forward

This has the same effect as manual block
execution in your current
shell.

`mdrip` accepts file names as directory arguments, and
runs them in the order specified.  Specify `README.md`
first, followed by file names as they appear in the
specified by `README_ORDER.txt` files, recursively
descending as needed, stopping where desired.

To go from zero to the point where one pod runs, try this:

```
eval "$(mdrip --label test \
    ./README.md \
    ./environment.md \
    ./hosting/minikube.md \
    ./hosting/confirm.md \
    ./hello.md \
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

#### Printing

As implied by the above command, the
following generates one bash script holding
all blocks with the `@test` label:

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

## Late-start testing

When _debugging blocks in test mode_, it's sometimes
desirable to run some part of the tutorial in test
mode without rerunning everything from the start.

But test mode is a subprocess. If not re-run from the
start, its environment won't be established.

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


## Early-stop testing

Just explicitly specify desired markdown files (in
proper order), rather than specifying the containing
directory:

```
mdrip \
    --mode test --label test --blockTimeOut 15m \
    ./README.md \
    ./environment.md \
    ./hosting/minikube.md \
    ./hosting/confirm.md \
    ./hello.md \
    ./containerize.md \
    ./review/namespaces.md \
    ./review/pods.md  # stop whereever
```
