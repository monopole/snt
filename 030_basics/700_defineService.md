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
  (the same port number on every node) into your
  service.  The IP must be discovered by inspection.
  Information about it appears in
  `<NodeIP>:spec.ports[*].nodePort` and
  `spec.clusterIp:spec.ports[*].port`.

* The example below uses `LoadBalancer`, which will
  round-robin requests to backend pods.  Balancer
  creation is (like most other things) asynchronous.
  Information about it appears in the
  `status.loadBalancer` field.

<!-- @defineFunctionToCreateService -->
```
function tut_CreateService {
cat <<EOF | kubectl create -f -
kind: Service
apiVersion: v1
metadata:
  name: $TUT_SERVICE_NAME

spec:
  #  Which pods to map to.
  selector:
    app: $TUT_APP_LABEL

  type: LoadBalancer

  # Traffic for the service is routed to the following ports.
  ports:
    - protocol: TCP

      # Where this service presents itself the external internet.
      port: $TUT_EXT_PORT

      # Set this to match the port believed to be in use on the pods.
      targetPort: $TUT_CON_PORT_VALUE
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
kubectl describe service $TUT_SERVICE_NAME
```

Crucial aspects of the output are

* Endpoints - the cluster addresses of the services the
  loadbalancer will talk to.  If this isn't set, then
  no pods are running with the given labels, or the
  pod targetPorts are wrong.
* Ingress - the external IP of the service.

Grab the load balancer's ingress IP address:

<!-- @hackToDetermineWhichAddressToUse -->
```
tmpl='{{ with index .items 0}}{{.metadata.name}}{{end}}'
firstNodeName=$(kubectl get -o go-template="$tmpl" nodes)
echo $firstNodeName
```

<!-- @defineFunctionToSetLbAddressVar -->
```
function tut_SetLbAddressVar {
  echo "This takes around 30 sec after service creation to work."
  if [ "$firstNodeName" == "minikube" ]; then
    local tmpl='{{.spec.clusterIP}}'
  else
    local tmpl='{{range .status.loadBalancer.ingress -}}{{.ip}}{{end}}'
  fi
  TUT_LB_ADDRESS=""
  while [ -z "$TUT_LB_ADDRESS" ]; do
    TUT_LB_ADDRESS=$(kubectl get -o go-template="$tmpl" service $TUT_SERVICE_NAME)
    if [ -z "$TUT_LB_ADDRESS" ]; then
      echo "waiting"
      sleep 2
    fi
  done
}
```

<!-- @setLoadBalancerAddressVar -->
```
tut_SetLbAddressVar
```

<!-- @viewLoadBalancerAddressVar -->
```
echo "Service at $TUT_LB_ADDRESS"
```

<!-- @defineFunctionToQueryServer -->
```
function tut_Query {
  local cmd="curl -m 1 $TUT_LB_ADDRESS:$TUT_EXT_PORT/$1"
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

The LB IP address also appears on the [address list
page] of your developer console, and in the entire
cluster's information dump:

<!-- @dumpClusterInfo -->
```
kubectl cluster-info dump | more
```
