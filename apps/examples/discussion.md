# Discussion

### Same result

Helm, DAM and the baseline approach all produced
instances of an app, and all supported the notion of
upgrade and rollback.

### App directory

Whereas the baseline approach doesn't prevent one from
arranging k8s resource files in a directory with a
manifest and calling it an _app_, helm and DAM (via
kinflate) require this.

### Instance control

The baseline and DAM approach rely on labels to
collectively operate on an instance's resource set.
helm tracks the notion of an app instance (it calls
this a _release_) via its server component called
_tiller_.

### App store

Whereas baseline k8s and kinflate don't prevent one
from treating github as an app store (using `git clone`
to download the app), helm builds this notion into its
client.

### The helm API

[power]: https://golang.org/pkg/text/template

A helm template (chart) author can bring the full
[power] of Go templates - branching, loops, callouts to
custom Go functions, etc - to creating k8s resources.
This ability, plus the new nouns and verbs offered by
helm+tiller, effectively create a new API to learn in
addition to the k8s API.  Users must respond to changes
as both APIs evolve in time.

### Version control

With either helm or baseline k8s, its up to a user to
impose their own philosophy about the relationship
between an "app store", forks of it under
version-control, forks of the app forks called
instances, and state in the cluster.  DAM anticipates
and encourages a fork and rebase approach (TODO: demo).
