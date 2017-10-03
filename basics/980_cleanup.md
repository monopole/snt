# Clean up

Either add an address record, e.g.

```
A * $TUT_STATIC_IP_NAME
```

to some naming service (e.g. namebright.com) then check it with

```
nslookup scriptingandtooling.com ns1.namebrightdns.com
```

or delete the static IP so we stop paying for it:

<!-- @deleteStaticIp -->
```
#  Obviously don't do this if you've recorded it in DNS records.
gcloud compute --project=$TUT_PROJECT_ID  \
    addresses delete $TUT_STATIC_IP_NAME
```

Also clean up other stuff that afaik isn't garbage collected:

<!-- @deleteConfigMap -->
```
kubectl delete configmap $TUT_CFG_NAME_1
```

<!-- @deleteDeployment -->
```
kubectl delete deployment $TUT_DEPLOY_NAME
```

Delete the cluster so we stop paying for it:

<!-- @deleteCluster -->
```
gcloud --quiet container clusters delete $TUT_CLUSTER_NAME
```
