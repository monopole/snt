# App Configuration Examples

> _Three ways to make the same cluster app._
>
> _Time: 10m_

The following sections describe making a set of instances
of the tuthello app using three different approaches:

 * baseline - use no special tools other than posix coreutils.
 * helm
 * Declarative Application Management

## The Common Scenario

In each example, the goal is to have two different
instances of the `tutHello` app co-existing in the
cluster at the same time, each running with their own
sets of pods:

 * Create an app instance called _production_.
 * Create a different instance called _staging_.
 * Once QA completes on _staging_, upgrade _production_
   to match it.  This is the rollout.
 * Users report a bug in production. Do a rollback.

Subsequent steps (_add a test to repro the bug, change
code to make test pass, push to staging, get QA
approval, push to prod, then continue the cycle with a
new release to staging, etc._) won't be done, as no new
concept is needed.

## The Common Ingredients

Each instance will need

 * a deployment resource,
 * a service resource (to tie a port to labelled pods),
 * configuration - the container image to run, the
   configMap resource to use, and whatever else needs
   to change in the deployment.

Instance resources are partitioned in the cluster via
_labels_.  The differences between using labels and
namespaces to partition resources was discussed
[earlier](/review/namespaces).
