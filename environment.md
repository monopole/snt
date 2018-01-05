# Environment

> _Define the tutorial environment._
>
> _Time: 30sec_


## Shell state

Global shell variables and functions
start with `TUT_` or `tut_` (an informal namespace).

For example, the following stores the initial _exit on
error_ behavior in a variable, and defines a function
to reset this behavior after running a command block.

This is crucial for placing blocks under CI/CD testing.

<!-- @exitOnErrStatus @env @test -->
```
export TUT_EXIT_ON_ERR=0
if [[ "$SHELLOPTS" =~ "errexit" ]]; then
  TUT_EXIT_ON_ERR=1
fi

function tut_restoreErrorOnExit {
  if [ "$TUT_EXIT_ON_ERR" -eq 1 ]; then
    # Normal for batch testing.
    set -e
  else
    # Normal for interactive use.
    set +e
  fi
}
```

## Workspace

Create a disposable working directory.

Use a specifically named directory rather than randomly
named directory (`mktemp -d`) to ease restarting
the tutorial at different points.

<!-- @defTmpDir @env @test -->
```
export TUT_DIR=$TMPDIR/k8s_config_tutorial
```

Optionally clear it:

<!-- @resetTmpDir @test -->
```
if [ -d "$TUT_DIR" ]; then
  /bin/rm -rf $TUT_DIR
fi
mkdir -p $TUT_DIR
```

All file system use happens in this disposable
directory.

```
echo $TUT_DIR
ls $TUT_DIR
```

Cleanup is just

> ```
> rm -rf $TUT_DIR
> ```

which your OS will eventually do on its own.
