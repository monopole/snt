# Kubernetes

This site is an experiment for a kubernetes tutorial.

It's served from a k8s cluster
(severe overkill) running this command
on its replicas:

```
mdrip --mode web gh:monopole/snt
```

I.e. the pods run [mdrip](http://github.com/monopole/mdrip)
in web server mode, serving content from
[github.com/monopole/snt](http://github.com/monopole/snt).
