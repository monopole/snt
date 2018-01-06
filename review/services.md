# Expose the Pod With a Service

> _Make pod services available outside the cluster._
>
> _Time: 5m_

Pods are ephemeral by design; they are expected to
crash or be killed, then be reanmimated.

Their cluster IP and port change as they come and go,
but their k8s _labels_ stay the same.  To reliably
contact them, one must _select them by label_.

In k8s, the thing that does this is called a [Service].
A k8s service is a pod label selector with a _fixed_ IP
and port.

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

* The example below uses `LoadBalancer`, which
  round-robins requests to pods with the right label.
  Information about it appears in the
  `status.loadBalancer` field.

<!-- @funcCreateService @env @test -->
```
function tut_createService {
cat <<EOF | kubectl apply -f -
kind: Service
apiVersion: v1
metadata:
  name: svc-eggplant

spec:
  #  Which pods to map to.
  selector:
    app: tuthello

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

<!-- @createService @test -->
```
tut_createService
```

Check again:

<!-- @getService @test -->
```
kubectl get service
```

Find out what's going on with the service:


<!-- @describeService @test -->
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

Obtain the service's address:

<!-- @funcGetAddress @env @test -->
```
function tut_getServiceAddress {
  local name=$1
  if tut_isMinikube; then
    local tm='{{range .spec.ports -}}{{.nodePort}}{{end}}'
    local nodePort=$(kubectl get -o go-template="$tm" \
        service $name)
    echo $($MINIKUBE_HOME/minikube ip):$nodePort
  else
    # Running on GKE presumably
    local tm='{{range .status.loadBalancer.ingress -}}{{.ip}}{{end}}'
    local lbAddress=""
    while [ -z "$lbAddress" ]; do
      lbAddress=$(kubectl get -o go-template="$tm" \
          service $name)
      if [ -z "$lbAddress" ]; then
        sleep 2
      fi
    done
    echo lbAddress:8666
  fi
}
```

Make a function to query the service with curl:

<!-- @funcQueryServer @env @test -->
```
function tut_query {
  local addr=$(tut_getServiceAddress $1)
  local path=$2
  # See curl exit codes at
  # https://curl.haxx.se/libcurl/c/libcurl-errors.html
  tut_retry 5 curl --fail --silent --max-time 3 $addr/$path
}
```

Try it, with different argument to see their role:

<!-- @queryService @test -->
```
tut_query svc-eggplant banana
tut_query svc-eggplant tangerine
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

Delete the pod to avoid confusion in what follows:

<!-- @deletePod @test -->
```
kubectl delete pod pod-tomato
```
