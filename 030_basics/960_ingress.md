# Ingress and DNS

The TCP loadbalancer user here, despite serving HTTP in
examples above, is [not designed] to terminate [TLS]
(transport layer security) connections.  Also, it isn't
monitored for health (?).

[not designed]: https://cloud.google.com/container-engine/docs/tutorials/http-balancer

[TLS]: https://cloud.google.com/compute/docs/load-balancing/http/#tls_support

Properly facing the web requires an ingress resource at a static
IP.  They cost about 25 cents per day, even when the cluster is
not running.  Create one:

<!-- @createStaticIP -->
```
# Harmless if the IP (with the given name) was already reserved.
gcloud compute --project=$TUT_PROJECT_ID \
  addresses create $TUT_STATIC_IP_NAME \
  --description=for\ scriptingandtooling.com \
  --global
```

<!-- @describeStaticIP -->
```
gcloud compute addresses describe --global $TUT_STATIC_IP_NAME
```

List known addresses:

<!-- @listStaticIP -->
```
gcloud compute addresses list
```

Create an associated ingress:

<!-- @createIngress -->
```
cat <<EOF | kubectl create -f -
kind: Ingress
apiVersion: extensions/v1beta1
metadata:
  name: $TUT_INGRESS_NAME
  annotations:
    kubernetes.io/ingress.global-static-ip-name: $TUT_STATIC_IP_NAME
spec:
  backend:
    serviceName: $TUT_SERVICE_NAME
    servicePort: $TUT_EXT_PORT
EOF
```

<!-- @describeIngress -->
```
kubectl describe ingress $TUT_INGRESS_NAME
```

<!-- @captureStaticIP -->
```
TUT_STATIC_IP_VALUE=$(gcloud compute addresses \
  describe $TUT_STATIC_IP_NAME \
  --global --format='value(address)')
echo $TUT_STATIC_IP_VALUE
```

<!-- @echoStaticIP -->
```
echo $TUT_STATIC_IP_VALUE
```

Now hit the ingress point:

<!-- @hitIngress -->
```
curl -m 1 $TUT_STATIC_IP_VALUE
```

<!-- @hitIngressWithExplicitProtocolAndPort -->
```
curl -m 1 http://$TUT_STATIC_IP_VALUE:80
```
