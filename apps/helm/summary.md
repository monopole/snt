# Summary

 * helm produced the same result as the [earlier
   exercise](/review/configuration/lifecycle) using raw
   kubectl apply commands.

 * Whereas raw k8s doesn't prevent one from arranging
   k8s resource files in a directory with a manifest and
   calling it an _app_, helm requires this.

 * Whereas raw k8s has a means to collectively operate
   on an app's resource set (via labels), helm enforces
   a notion of a atomic app since its resource set is
   created, tracked and deleted as a unit by _tiller_.

 * Whereas raw k8s doesn't prevent one from treating
   github as an app store and downloading named resource
   bundles from a repo, helm builds this notion into its
   client.

 * With either helm or native k8s, its up to a user to
   impose their own philosophy about the relationship
   between source controlled files and instance state in the cluster.

[power]: https://golang.org/pkg/text/template

A helm template author can
bring the full [power] of Go templates -
branching, loops, callouts to custom Go functions,
etc - to creating k8s resources.

It's up to a user to decide how much of their deployment
process they wish to express via Go template features.

[approach]: /review/configuration/lifecycle

For a simple case like `tuthello`, a syntax and
type-unaware [approach] using _sed_ proved sufficient.