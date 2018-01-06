# Ingress

> _Make your service available worldwide._
>
> _Time: 5min_


__Skip this if using minikube - this section about
exposing a service from GKE to the outside world__

__Warning: Not under CI/CD testing (due to auth and billing issues).__

The loadbalancer defined in the
[services](/review/services) section is [not designed]
to terminate [TLS] connections.


[not designed]: https://cloud.google.com/container-engine/docs/tutorials/http-balancer
[TLS]: https://cloud.google.com/compute/docs/load-balancing/http/#tls_support

Properly facing the web requires an _ingress resource_
at a static IP.  They cost about 25 cents per day, even
when the cluster is not running.

Create one:

<!-- @env -->
```
TUT_STATIC_IP_NAME=peach-static-ip
```

Harmlessly see if this IP name was already reserved.

<!-- @createStaticIP -->
```
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

Create an associated ingress, pointing it at `svc-eggplant`
(created in the [services](/review/services) section):

<!-- @createIngress -->
```
cat <<EOF | kubectl create -f -
kind: Ingress
apiVersion: extensions/v1beta1
metadata:
  name: ingress-broccoli
  annotations:
    kubernetes.io/ingress.global-static-ip-name: $TUT_STATIC_IP_NAME
spec:
  backend:
    serviceName: svc-eggplant
    servicePort: 8666
EOF
```

<!-- @describeIngress -->
```
kubectl describe ingress ingress-broccoli
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

<!-- @hitIngress1 -->
```
curl -m 1 $TUT_STATIC_IP_VALUE
```

<!-- @hitIngress2 -->
```
curl -m 1 http://$TUT_STATIC_IP_VALUE:80
```

## Cleanup

At this point you can delete the static IP (and stop paying for it):

<!-- @deleteStaticIp -->
```
gcloud compute --project=$TUT_PROJECT_ID  \
    addresses delete $TUT_STATIC_IP_NAME
```

Or you can join the internet, i.e. add an address record

```
A * $TUT_STATIC_IP_NAME
```

to some DNS, e.g. [namebright](https://namebright.com).

See if the name you want is open:

```
nslookup your-new-website.com ns1.namebrightdns.com
```
