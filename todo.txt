# todo

#### gcloud install isolation

see if gcloud installation can be constrained to TUT_DIR

#### support OSX

use env var USING_MAC_OSX to provide alternative to linux

#### better distinguish minikube path from GKE

another 'big' env var

#### add one sentence summaries to each page

Along with a time estimate.  this was done in configmaps.

#### find home for this?


> To learn what fields are
> available for printing via Go templates, look at the
> underlying yaml:
>
> <!-- @getNodeYaml -->
> ```
> kubectl get -o yaml nodes
> ```
>
> To make data available in a shell loop (e.g. to use it
> as an arg to `curl`):
>
> <!-- @nodeDataToCurl -->
> ```
> nodes=$(kubectl get \
>   -o go-template="{{range .items}}{{.metadata.name}} {{end}}" \
>   nodes)
> for n in $nodes; do
>   echo $n
> done
> ```
>
> As an example of filtering output
> from a list, grab IP data from the nodes:
>
> <!-- @getNodeIps -->
> ```
> function getIps {
>   local tmpl=`cat <<EOF
> {{range .items -}}
> {{\\\$n := .metadata.name -}}
>   {{range .status.addresses -}}
>     {{if eq .type "$1"}}{{\\\$n}} {{.address}}{{end -}}
>   {{end}}
> {{end}}
> EOF
> `
>   kubectl get -o go-template="$tmpl" nodes
> }
> getIps InternalIP
> getIps ExternalIP
> ```
