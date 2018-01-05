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
  (server, prober, monitor, static data set).

* It offers a versioned _API_ allowing other server sets
  to talk to it per compatibility and deprecation
  policies.

* It co-exists on its host with other server sets,
  e.g. mail servers.

* It may be unaware of these sets, or it may need to
  discover and depend on them, e.g. satellite photo
  servers, road traffic update servers.

* The release lifecycle of these other sets is
  unorchestrated, and performed by different teams.

For simplicity, the following refers to these sets
as _cluster apps_.

## Kubernetes Apps are a bundle of YAML

Per the [review](/review), a cluster app is -
minimally - the set of all YAML files needed to create
the k8s resources ([services](/review/services),
[deployments](/review/deployment), [config
maps](/review/configMap), etc.) that assure the
associated service set runs in the cluster.

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