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
export TUT_DIR=$HOME/k8s_config_tutorial
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
  local limit=$1
  local k=1
  shift
  local cmd="$@"
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
