# Environment

> _Define the tutorial environment._
>
> _Time: 30s_

Global shell variables and functions
start with `TUT_` or `tut_` (an informal namespace).

All file system use is confined to the
disposable directory defined in `TUT_DIR`:

<!-- @defineIt @env @test -->
```
export TUT_DIR=$TMPDIR/k8s_config_tutorial
export TUT_TMP=$TUT_DIR/tmp
export TUT_BIN=$TUT_DIR/bin
PATH=$TUT_BIN:$PATH
```

<!-- @optionallyClearIt @test -->
```
if [ -d "$TUT_DIR" ]; then
  /bin/rm -rf $TUT_DIR
fi
```

<!-- @makeIt @test -->
```
mkdir -p $TUT_TMP $TUT_BIN
```

Cleanup is just

> ```
> rm -rf $TUT_DIR
> ```

which your OS will eventually do on its own.

## Tutorial error handling

In CI/CD [testing](/appendix/testing) of this tutorial,
one wants a command block error to cause process exit
with a reportable error code.  In interactive use, one
obviously doesn't want that because it closes the
terminal.

For the benefit of either scenario, the following
stores the initial _exit on error_ behavior in a
shell variable and defines a function to reset it.

<!-- @exitOnErrStatus @env @test -->
```
export TUT_EXIT_ON_ERR=0
if [[ "$SHELLOPTS" =~ "errexit" ]]; then
  TUT_EXIT_ON_ERR=1
fi

function tut_restoreErrorOnExit {
  if [ "$TUT_EXIT_ON_ERR" -eq 1 ]; then
    # Test behavior.
    set -e
  else
    # Interactive behavior.
    set +e
  fi
}
```
## Retry

Clust manipulation commands are asynchronous; they
don't wait for the cluster to 'obey' the command before
returning.

Subsequent actions may need to wait, or more robustly,
be prepared to retry.  The following count-limited
retry function is used through the tutorial.

<!-- @funcRetry @env @test -->
```
function tut_retry {
  local limit=$1
  shift
  local cmd="$@"
  local k=1
  set +e
  while [ $k -le $limit ]; do
    $cmd
    local code=$?
    if [ $code -eq 0 ]; then
      tut_restoreErrorOnExit
      return
    fi
    printf "Attempt $k/$limit exitted with $code.  "
    k=$((k + 1))
    if [ $k -le $limit ]; then
      echo "Retrying..."
      sleep 2;
    fi
  done
  printf "\nFailed: $cmd \n"
  tut_restoreErrorOnExit
  /bin/false # Fail.
}
```
