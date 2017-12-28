# Expose the Pod With a Service

> _Make pod services available outside the cluster._
>
> _Time: 5min_

Pods are not exposed to the outside world directly.  To
access a service on a pod, one  needs a k8s [Service] -
providing a fixed (albeit perhaps ephemeral) IP and a
static port.

[Service]: https://kubernetes.io/docs/concepts/services-networking/service

Confirm that there are no services:

<!-- @getService @test -->
```
kubectl get service
```

Create one in the form of a loadbalancer.

The `spec`'s `type` field invokes different
behaviors:

* `ClusterIP`: The default, means a service
  visible only to other pods in the cluster - no
  external access.

* `NodePort`: The k8s master will allocate a port that
  is unused on all nodes, and have the nodes forward
  from that port to the service.  The IP must be
  discovered by inspection.  Information about it
  appears in `<NodeIP>:spec.ports[*].nodePort` and
  `spec.clusterIp:spec.ports[*].port`.

* The example below uses `LoadBalancer`, which will
  round-robin requests to pods.  Balancer creation is
  (like most other things) asynchronous.  Information
  about it appears in the `status.loadBalancer` field.

<!-- @funcCreateService @test @debug -->
```
function tut_CreateService {
cat <<EOF | kubectl apply -f -
kind: Service
apiVersion: v1
metadata:
  name: svc-eggplant

spec:
  #  Which pods to map to.
  selector:
    app: avocado

  type: LoadBalancer

  # Traffic for the service is routed to the following ports.
  ports:
    - protocol: TCP

      # Where this service presents itself the external internet.
      port: 8666

      # The port at which the user's software in the container
      # listens for requests.
      targetPort: 8080
EOF
}
```

<!-- @createService @test @debug -->
```
tut_CreateService
```

Check again:

<!-- @getService @test @debug -->
```
kubectl get service
```

Find out what's going on with the service:


<!-- @describeService @test @debug -->
```
kubectl describe service svc-eggplant
```

Crucial aspects of the output are

* Endpoints - the cluster addresses of the services the
  loadbalancer will talk to.  If this isn't set, then
  no pods are running with the given labels, or the
  pod targetPorts are wrong.
* Ingress - the external IP of the service;
  not present in minikube.

Grab an address to use with the service:

<!-- @funcGetAddress @test @debug -->
```
function tut_getServiceAddress {
  if tut_isMinikube; then
    local tmpl='{{range .spec.ports -}}{{.nodePort}}{{end}}'
    local nodePort=$(kubectl get -o go-template="$tmpl" service svc-eggplant)
    echo $($MINIKUBE_HOME/minikube ip):$nodePort
  else
    # Running on GKE presumably
    local tmpl='{{range .status.loadBalancer.ingress -}}{{.ip}}{{end}}'
    local lbAddress=""
    while [ -z "$lbAddress" ]; do
      lbAddress=$(kubectl get -o go-template="$tmpl" service svc-eggplant)
      if [ -z "$lbAddress" ]; then
        sleep 2
      fi
    done
    echo lbAddress:8666
  fi
}
```

<!-- @getAddress @test @debug -->
```
echo "This may take around 30 sec after service creation to work."
TUT_SVC_ADDRESS=$(tut_getServiceAddress)
echo "Service at $TUT_SVC_ADDRESS"
```

<!-- @funcQueryServer @test @debug -->
```
function tut_Query {
  curl --fail --silent --max-time 3 $TUT_SVC_ADDRESS/$1
}
```

<!-- @queryServiceRaw1 @test @debug -->
```
tut_Query banana
```

<!-- @queryServiceRaw2 @test @debug -->
```
tut_Query tangerine
```

If running on GKE, The LB IP address also appears on the
[address list page](https://console.cloud.google.com/networking/addresses/list)
of your developer console, and in the entire cluster's
information dump:

<!-- @dumpClusterInfo -->
```
if ! tut_isMinikube; then
  kubectl cluster-info dump | grep Ingress
fi
```
