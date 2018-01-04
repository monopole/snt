# Cluster App Configuration

[Google Maps API]: https://enterprise.google.com/maps/products/mapsapi.html
[third-party]: https://www.noupe.com/development/collection-of-the-coolest-uses-of-the-google-maps-api.html

The [Google Maps API] is, most literally, a _set of
replicated frontend servers_ running on Google's
clusters.  It backs an enormous number of
[third-party] services.

Despite its size and complexity, this set has
properties like a phone app.

* It has a singular description and purpose -
  e.g. _Serve the Maps API_.

* It can be installed and deleted.

* It has a lifecycle of development, test and release,
  with docs and release notes.

* It has a versioned implementation - albeit likely
  associated with a _list_ of versions of its components
  (servers, probers, monitors, static data).

* It may offer a versioned API, independent of the
  implemention version, with a deprecation and version
  skew compatibility policy.

* It co-exists on its hardware with other server sets,
  e.g. email servers, photo servers.

* It may be unaware of these sets, or it may need to
  discover and depend on them.

In particular, a maps API depends on distributed
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

As covered in the [review](/k8sReview), this is just a
collection of one or more yaml files, hopefully stored
in a version control system.  The yaml files are
recipes, and the container images pulled from some
registry are the ingredients.  The set of yaml files
looks even more like an app if one bundles them with a
manifest file listing them and describing their overall
purpose.

[outside]: https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/#what-kubernetes-is-not

From there one could build a tool to discover, perhaps
buy, and install such bundles - creating yet another app
ecosystem. That's [outside] the purview of the core
kubernetes project.

The rest of this tutorial focusses on the important
problem of managing different instances of an app.
