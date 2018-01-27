# Tutorial Testing

> _How to test or rerun all or part of the tutorial._
>
> _Time: ~5min_

## Prerequisites

[mdrip]: https://github.com/monopole/mdrip

Tutorial testing uses [mdrip] to rip command blocks
from markdown for execution.

<!-- @installMdrip -->
```
if [ -z ${TMPDIR+x} ]; then TMPDIR=/tmp; fi
GOBIN=$TMPDIR go install github.com/monopole/mdrip
alias mdrip=$TMPDIR/mdrip
```

The tutorial content lives on github.  Clone it:

<!-- @installContent -->
```
TUT_CONTENT=$TMPDIR/tutorial
/bin/rm -rf $TUT_CONTENT
git clone https://github.com/monopole/snt $TUT_CONTENT
```


## Test the entire tutorial

<!-- @testAllContent -->
```
mdrip --mode test --label test \
    --blockTimeOut 15m $TUT_CONTENT
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
mdrip --label test $TUT_CONTENT | more
```
