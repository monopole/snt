# Make a Single-Component App

> _Write a configurable HTTP server
>  to run on the cluster._
>
> _Time: 30sec_

A cluster-wide app may involve any number of
components - web servers, log capturers, databases,
etc.  This tutorial will use just one component, with
the minimal complexity needed demonstrate
configuration, rollout, rollback, upgrade, etc.

The program will serve a tiny web page
whose appearance is influenced by flags and
shell env variables.

To demo rollouts and rollbacks, there will be
multiple versions of the program, with the
version appearing on the served web page.


<!-- @defineEnv -->
```
TUT_IMG_NAME=radishwine
TUT_IMG_V1=1  # to tag version 1
TUT_IMG_V2=2  # to tag version 2
TUT_IMG_PATH=$TUT_DIR/src/$TUT_IMG_NAME

mkdir -p $TUT_DIR/src
```

<!-- @makeWebServer -->
```
cat <<'EOF' >${TUT_IMG_PATH}.go
package main

import (
    "flag"
    "github.com/golang/glog"
    "html/template"
    "net/http"
    "os"
    "strconv"
    "time"
)

type config struct {Risky bool; Port int; Greeting string}
type server struct {C *config; P string; V int}

const (
    version = 0
    tmplName = "whatever"
    tmplBody = `
{{define "` + tmplName + `" -}}
<html><body>
Version {{.V}} : {{if .C.Risky}}<em>{{end}}
{{- .C.Greeting}}{{if .C.Risky}}</em>{{end}} {{.P}}
</body></html>
{{end}}
`)

var (
    enableRiskyFeature = flag.Bool("enableRiskyFeature", false,
        "Enables some risky feature.")
    port = flag.Int("port", 8080, "Port at which HTTP is served.")
)

func getConfig() *config {
    flag.Parse()
    greeting := os.Getenv("TUTORIAL_GREETING")
    if len(greeting) == 0 {
        greeting = "Hello"
    }
    return &config{*enableRiskyFeature, *port, greeting}
}

func main() {
    c := getConfig()
    tmpl := template.Must(template.New("main").Parse(tmplBody))
    http.HandleFunc("/quit",
      func (w http.ResponseWriter, r *http.Request) {
        go func() {time.Sleep(1 * time.Second); os.Exit(0)}()
      })
    http.HandleFunc("/",
      func (w http.ResponseWriter, r *http.Request) {
        if err := tmpl.ExecuteTemplate(
          w, tmplName, &server{c, r.URL.Path[1:], version});
          err != nil {
          glog.Fatal(err)
        }
      })
    hostPort := ":" + strconv.Itoa(c.Port)
    if err := http.ListenAndServe(hostPort, nil); err != nil {
        glog.Fatal(err)
    }
}
EOF
```

Build version 1 of the program to see that it works.

<!-- @funcToBuild -->
```
# Builds binary with a hard-coded version stamp.
function tut_BuildProgram {
  local version=$1
  local src=${TUT_IMG_PATH}${version}.go
  cat ${TUT_IMG_PATH}.go \
      | sed 's/version = 0/version = '${version}'/' > $src
  CGO_ENABLED=0 GOOS=linux \
      go build -o $TUT_IMG_PATH -a -installsuffix cgo $src
}
```

<!-- @buildAtV1 -->
```
rm -f $TUT_IMG_PATH
tut_BuildProgram $TUT_IMG_V1
file $TUT_IMG_PATH
ls -sh $TUT_IMG_PATH
```

<!-- @funcRunAndKill -->
```
function tut_RequestAndQuit {
  local port=$1
  local path=$2
  # Give server time to fire up.
  sleep 2
  # Dump webpage to stdout
  curl -m 1 localhost:$port/$path
  # Send query of death
  curl -m 1 localhost:$port/quit
}
```

<!-- @runAndKill -->
```
TUTORIAL_GREETING=salutations ${TUT_IMG_PATH} \
    --enableRiskyFeature --port 8100 &
tut_RequestAndQuit 8100 godzilla
```

The server works.  It just needs a place to run.
