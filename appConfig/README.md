# App Level Configuration

People access Amazon, Facebook, Google etc. through
mobile or web apps that talk to vast clusters running
some service that also has app-like properties.

This cluster-level service app, talking to billions of
phones or web browsers, updates periodically, and if
something goes wrong is rapidly replaced with its older
version.  There's a dedicated team that maintains and
improves this app.

Further, this cluster-level app talks to a plethora of
other cluster-level apps, all evolving on their own
lifecycle.

For example, Google Maps is backed by
services dedicated to serving map tiles, satellite
photos, restaurant reviews, providing translations,
collecting live traffic status, storing user
preferences, etc.

These other cluster-level apps are maintained by
completely different teams that do not, and indeed
cannot, attempt to orchestrate any kind of timing
around when they release new versions.  The apps
must be able to evolve without coordination.

## kubernetes Apps

In kubernetes, cluster-level apps are characterized by
the set of all k8s resources needed to serve them - the
ConfigMaps, the Deployments, etc.

As has been seen, this is just a collection of one or
more yaml files.  The yaml files are _recipes_, and the
container images themselves are the _ingredients_.
Where containers come from is a related, yet distinct,
concern.

## App Configuration

Any app in a cluster is likely to have many 
instances, differing on various dimensions:

* exposure: production, staging, QA, dev snapshot, ...
* regions: Asia, Europe, The Americas, ...
* privacy/security:  public, goverment, HIPPA, ITAR, ...
* tier: frontend, middle, backend, ...

Managing these instances is a configuration problem.

The following discusses two approaches to cluster-level
app configuration in k8s: _helm_ and _kinflate_.
