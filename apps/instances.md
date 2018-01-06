# Instance Configuration

A cluster app will have many instances, running at the
same time possibly in the same cluster, differing in
their configuration across various dimensions:

* reliability: production, staging, QA, dev snapshot, ...
* jurisdiction: China, EU, USA, ...
* security:  public, government, HIPPA, ITAR, ...
* age: latest, last week, last month, ...

These instances must be configured and managed.

## Imperative configuration

> _Use imperative commands -_ __kubectl create__,
> __kubectl run__, __kubectl delete__, _etc. - to get
> the instances into their desired state._

This works, but has a handoff problem.  If someone
gives you the keys to the cluster, how do you know
what's been done?  The only way forward is to query
things and try to issue more imperative commands.

It's best to eschew imperative commands entirely.

## Declarative configuration

> _Create a directory for each instance, copy all the
> YAML files into each directory, let the files evolve
> independently, and use_ __kubectl apply__ _after file
> changes._

This is much better. There are files that can be stored
in version control, and the declarative usage described
(`kubectl apply`) means one need merely _read_ the
YAML files to understand the state of the cluster.

One can verify this state with `kubectl diff`,
comparing the specification in the files to the actual
cluster state.

## The difference that matters

The new problem is that only some small percentage of
the YAML changes from one instance to the next.
Duplicating all the YAML introduces room for mistakes and
makes common changes hard.

The following discusses two approaches to solving
this problem - _helm_ and _kinflate_.
