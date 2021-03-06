# Configurable Hello World

> _Write a configurable HTTP server
>  to run on the cluster._
>
> _Time: 30s_

A cluster-wide app may involve any number of programs -
web servers, log capturers, databases, etc.

This tutorial will use just one program, called
`tuthello`, with the minimal complexity needed to
demonstrate configuration, rollout, rollback and
upgrade.

`tuthello` serves a tiny hello-world style web page
whose appearance is influenced by flags and shell
variables.  Further, there will be two distinct binary
versions of `tuthello` differing in a hardcoded
constant.

<!-- @env @test -->
```
TUT_IMG_PATH=$TUT_DIR/src/tuthello
```

<!-- @mkSrcDir @test -->
```
mkdir -p $TUT_DIR/src
```

<!-- @makeWebServer @test -->
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
    tmplName = "homepage"
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
    greeting := os.Getenv("ALT_GREETING")
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

This web server is configurable via

 * one compile-time constant: `version`
 * two flags: `--enableRiskyFeature`, `--port`
 * one environment variable: `ALT_GREETING`

It uses its URL query path in the greeting, or to quit.

Define a function to build the server, changing the
compile-time constant:

<!-- @funcToBuild @env @test -->
```
# Builds binary with a hard-coded version stamp.
function tut_buildProgram {
  local version=$1
  local src=${TUT_IMG_PATH}${version}.go
  cat ${TUT_IMG_PATH}.go \
      | sed 's/version = 0/version = '${version}'/' > $src
  CGO_ENABLED=0 GOOS=linux \
      go build -o $TUT_IMG_PATH -a -installsuffix cgo $src
}
```

Build it at v1.

<!-- @buildAtV1 @test -->
```
rm -f $TUT_IMG_PATH
tut_buildProgram 1  # version one
if ! type -P $TUT_IMG_PATH >/dev/null 2>&1; then
  echo Build failed?
fi
```

Run it and shut it down.

<!-- @runAndQuit @test -->
```
# Start server
ALT_GREETING=salutations \
    $TUT_IMG_PATH --enableRiskyFeature --port 8100 &

# Let it get ready
sleep 2

# Dump html to stdout
curl --fail --silent -m 1 localhost:8100/godzilla

# Send query of death
curl --fail --silent -m 1 localhost:8100/quit
```
