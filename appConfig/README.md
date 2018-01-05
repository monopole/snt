# Cluster App Configuration

[Google Maps API]: https://enterprise.google.com/maps/products/mapsapi.html

The [Google Maps API] is a set of replicated frontend
servers running on Google's clusters.  It backs many
Google and third-party services.

Despite its size and complexity, this set has
properties like a phone app.

* It has a singular description and purpose:
  _serve the Maps API_.

* It can be installed and deleted.

* It has a lifecycle of development, test and release,
  with docs and release notes.

* It has a versioned _implementation_ - albeit likely
  associated with a list of versions of its components
  (servers, probers, monitors, static data).

* It offers a versioned _API_ allowing other server sets
  to talk to it per compatibility and deprecation
  policies.

* It co-exists on its hardware with other server sets,
  e.g. email servers, photo servers.

* It may be unaware of these sets, or it may need to
  discover and depend on them.

The maps API depends on distributed services offering
map tiles, satellite photos, reviews, translations,
traffic updates, user preferences, etc.  These sets of
services evolve on uncoordinated lifecycles managed by
independent teams.

For simplicity, the following calls these sets
_cluster apps_.

## Kubernetes Apps are a bundle of YAML

Per the [review](/review), cluster apps are - minimally -
the set of all YAML files needed to create k8s
resources like [services](/review/services),
[deployments](/review/deployment), and [config
maps](/review/configMap).

The YAML files are recipes, and the container images
pulled from some registry are the ingredients.

[outside]: https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/#what-kubernetes-is-not

The set of YAML files looks even more like an app if
one bundles them with a manifest file listing them and
describing their overall purpose.  From there one could
build a tool to discover, perhaps buy, and install such
bundles - creating yet another app ecosystem. That's
[outside] the purview of the core kubernetes project.

The rest of this tutorial focusses on further defining
an app, guided by the need for different app
_instances_.
