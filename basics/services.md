### Expose the Pod With a Service

Pods are not exposed to the outside world directly.  To
access a service on a pod, one  needs a k8s [Service] -
providing a fixed (albeit perhaps ephemeral) IP and a
static port.

[Service]: https://kubernetes.io/docs/concepts/services-networking/service

First confirm that there are no services:

<!-- @getService -->
```
kubectl get service
```

Then create one in the form of a loadbalancer.

The `spec`'s `type` field invokes different
behaviors:

* The default is `ClusterIP`, which means a service
  visible only to other pods in the cluster - no
  external access.

* Using `NodePort` here means the Kubernetes master
  will allocate a port from a flag-configured range in
  the low 30,000s, and each node will proxy that port
  (the same port number on every node) to your
  service.  The IP must be discovered by inspection.
  Information about it appears in
  `<NodeIP>:spec.ports[*].nodePort` and
  `spec.clusterIp:spec.ports[*].port`.

* The example below uses `LoadBalancer`, which will
  round-robin requests to pods.  Balancer creation is
  (like most other things) asynchronous.  Information
  about it appears in the `status.loadBalancer` field.

<!-- @defineFunctionToCreateService -->
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
      port: 8088

      # Set this to match the port believed to be in use on the pods.
      targetPort: 8080
EOF
}
```

<!-- @createService -->
```
tut_CreateService
```

Check again:

<!-- @getService -->
```
kubectl get service
```

Find out what's going on with the service:
The output of this command includes an endPoint.

<!-- @describeService -->
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

<!-- @hackToDetermineWhichAddressToUse -->
```
```

<!-- @defineFunctionToGetServiceAddress -->
```
function tut_getServiceAddress {
  tmpl='{{ with index .items 0}}{{.metadata.name}}{{end}}'
  firstNodeName=$(kubectl get -o go-template="$tmpl" nodes)
  if [ "$firstNodeName" == "minikube" ]; then
    # Running on minikube
    local tmpl='{{range .spec.ports -}}{{.nodePort}}{{end}}'
    local nodePort=$(kubectl get -o go-template="$tmpl" service svc-eggplant)
    echo $(minikube ip):$nodePort
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
    echo lbAddress:8088
  fi
}
```

<!-- @setLoadBalancerAddressVar -->
```
echo "This may take around 30 sec after service creation to work."
TUT_SVC_ADDRESS=$(tut_getServiceAddress)
echo "Service at $TUT_SVC_ADDRESS"
```

<!-- @defineFunctionToQueryServer -->
```
function tut_Query {
  local cmd="curl -m 1 $TUT_SVC_ADDRESS/$1"
  echo $cmd
  $cmd
}
```

Hit your server:
<!-- @curlService -->
```
tut_Query bananna
```

[address list page]: https://console.cloud.google.com/networking/addresses/list

If running on GKE, The LB IP address also appears on
the [address list page] of your developer console, and
in the entire cluster's information dump:


<!-- @dumpClusterInfo -->
```
kubectl cluster-info dump |grep Ingress
```
