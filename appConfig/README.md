# Cluster App Configuration

People access Amazon, Facebook, Google etc. through
mobile or web apps supported by
services distributed on a cluster.

[Google Maps API]: https://enterprise.google.com/maps/products/mapsapi.html

This _set of distributed services_ has distinct
app-like properties:

* Despite its size and complexity, the set has some
  singular description and purpose - e.g. _Serve the
  [Google Maps API]_.

* It has an owner - e.g. a team of engineers, product
  managers, testers, etc.

* It has a lifecycle of development, testing, release
  with release notes and documentation.

* It has a versioned implementation, likely associated
  with a list of versions of its components, and likely
  expressed as a set of text files in a version control
  system.

In addition, the set will usually have

* A versioned API, distinct and independent of the
  implemention version, with a deprecation and version
  skew compatibility policy.

* An SLA.

Moreover

* The set may co-exist on a cluster with other sets.

* Theses sets may depend on each other, or be unaware
  of each other.

E.g the maps API is supported by sets of distributed
services offering map tiles, satellite photos, reviews,
translations, traffic updates, user preferences, etc.
These sets of services evolve on uncoordinated
lifecycles managed by independent teams.

For simplicity, the following calls these sets
_cluster apps_.

## Kubernetes Apps

In kubernetes, cluster apps are characterized by the
set of all k8s resources needed to serve them - the
ConfigMaps, the Deployments, etc.

As has been seen in the [review](/k8sReview), this is
just a collection of one or more yaml files.  The yaml
files are recipes, and the container images themselves
are the ingredients.  Where containers come from is a
related, yet distinct, concern.

## Instance Configuration

A cluster app will have many instances, differing on
various dimensions:

* quality: production, staging, QA, dev snapshot, ...
* regions: Asia, Europe, The Americas, ...
* security:  public, goverment, HIPPA, ITAR, ...
* tier: frontend, middle, backend, ...

Managing these instances is a configuration problem.

The following discusses two approaches to cluster
app configuration in k8s: _helm_ and _kinflate_.
