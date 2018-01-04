# Instance Configuration

A cluster app will have many instances, running at the
same time possibly in the same cluster, differing in
their configuration across various dimensions:

* reliability: production, staging, QA, dev snapshot, ...
* jurisdiction: China, EU, USA, ...
* security:  public, government, HIPPA, ITAR, ...
* age: latest, last week, last month, ...

How to maintain these instances?

## First attempt

> _Run kubectl imperative commands (create, run,
> delete, etc.) to get the instances into their desired
> state._

The reasons this is bad is left as an exercise for the reader.

## Second attempt

> _Create a directory for each instance,
> copy all the yaml files into each directory,
> let the files evolve independently, use_
> __kubectl apply__ _after changes._

This is much better. There are files that can be stored
in version control, and the usage described means the
files effectively _declare_ the cluster state. One
can check this with `kubectl diff`.

The new problem is that only some small percentage of
the yaml changes from one instance to the next.
Duplicating all of it introduces room for mistakes and
makes common changes hard.

The following discusses two approaches to solving
this problem - _helm_ and _kinflate_.
