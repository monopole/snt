# The base app

The manifest and its resources fully define a runnable app.

Try it:

<!-- @runKinflate @demo -->
```
kinflate -f $TUT_DAM/manifest |\
    kubectl apply -f -
```

<!-- @query @demo -->
```
tut-query tuthello peach
```
