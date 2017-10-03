# Multi-tenant Bursting on Kubernetes

Say you have multiple tenants on k8s where one (or
more) of the tenants may sustain a traffic sudden burst
(2X-3X) for a few hours (e.g. a flash sale).

One way to allow for this is pay for extra nodes that
can handle the burst but are otherwise idle.

The approach described below attempts to save money
by repurposing existing nodes away from their normal
duties to dedicate them to the burst, then post-burst
send them back to their normal work.
