# App Level Configuration

People access Amazon, Facebook, Google etc. through
mobile or web apps that talk to enormous computer
clusters running some service that also has app-like
properties.

This cluster-level service app, talking to billions of
phones or web browsers, updates periodically, and if
something goes wrong is rapidly replaced with its older
version.  There's a dedicated team that maintains and
improves this 'app'.

Further, this cluster-level app talks to a plethora of
other cluster-level apps, all evolving on their own
lifecycle.

For example, Google Maps is backed by services
dedicated to serving map tiles, satellite photos,
restaurant reviews, providing translations, collecting
live traffic status, storing user preferences, etc.

These other cluster-level apps are maintained by
completely different teams that do not, and indeed
cannot, attempt to orchestrate any kind of timing
around when they release new versions.

## kubernetes Apps

In kubernetes, cluster-level apps are characterized by
the set of all k8s resources needed to serve them - the
ConfigMaps, the Deployments, etc.

As has been seen, this is just a collection of one or
more yaml files.  The yaml files are recipes, and the
container images themselves are the ingredients - where
they come from is a related, yet distinct, concern.

## Making the App Concept Useful

We want to attach


Configuration -


In ka8s, configuration is

* exposure: production, staging, QA, dev snapshot,
* regions: Asia, Americas, Europe, ...
* privacy:  public, goverment, HIPPA, ITAR, ...
* tier: frontend, middle, back, ...

Managing these instances is a configuration problem.

The following discusses two approaches to cluster-level
app configuration in k8s: _helm_ and _kinflate_.
