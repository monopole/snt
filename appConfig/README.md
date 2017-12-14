# App Level Configuration

The definition of a cluster app could be _the set of
all things needed to provide some cluster-backed
service_.

People access Amazon, Facebook, Google etc. through
apps (mobile or web) that are backed by enormous clusters.

The many services backing user visible apps are
themselves cluster-level apps with independent
lifecycles, e.g. Google's Spanner is a cluster-level
storage app that backs many other cluster-level apps,
e.g. maps and ad serving.

A cluster app may have instances on many dimensions:

* exposure: production, staging, QA, dev snapshot,
* regions: Asia, Americas, Europe, ...
* privacy:  public, goverment, HIPPA, ITAR, ...
* tier: frontend, middle, back, ...

Managing these instances is a configuration problem.

The following discusses two approaches to cluster-level
app configuration in k8s: _helm_ and _kinflate_.
