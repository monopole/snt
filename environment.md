# Environment

> _Define the tutorial environment._
>
> _Time: 30s_

Global shell variables and functions
start with `TUT_` or `tut_` (an informal namespace).

All file system use is confined to the following
disposable directory:

<!-- @defineIt @env @test -->
```
export TUT_DIR=$TMPDIR/k8s_config_tutorial
```

<!-- @optionallyClearIt @test -->
```
if [ -d "$TUT_DIR" ]; then
  /bin/rm -rf $TUT_DIR
fi
```

<!-- @makeIt @test -->
```
mkdir -p $TUT_DIR
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
obviously doesn't want that (it closes the terminal).

For the benefit of either scenario, the following
stores the initial _exit on error_ behavior in a
shell variable and defines a function to reset it.
The latter is used in a retry function that provides
some flexibility throughout the tutorial.

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

function tut_retry {
  local code=0
  local i=$1
  shift
  local cmd="$@"
  set +e
  while [ $i -gt 0 ]; do
    $cmd
    code=$?
    if [ $code -eq 0 ]; then
      tut_restoreErrorOnExit
      return
    fi
    i=$((i - 1))
    if [ $i -gt 0 ]; then
      echo "Exit status $code - retrying..."
      sleep 2;
    fi
  done
  echo Failed with ${code}: $cmd
  tut_restoreErrorOnExit
  /bin/false # Fail.
}
```
