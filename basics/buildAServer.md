# Make a server

Make a program to use as an example binary to
configure, rollout, rollback, upgrade, etc.  Make it a
webserver so it can be queried with HTTP clients you
already have.

The server serves a simple web page whose appearance is
influenced by flags and shell env variables.

Additionally, to demo rollouts and rollbacks, there
will be multiple versions of the binary. The version
will also appear on the served web page.


<!-- @defineEnv -->
```
TUT_IMG_NAME=radishwine
TUT_IMG_V1=1
TUT_IMG_V2=2
TUT_IMG_PATH=$TUT_DIR/src/$TUT_IMG_NAME

mkdir -p $TUT_DIR/src
```

<!-- @makeWebServer -->
```
cat <<'EOF' >${TUT_IMG_PATH}.go
package main

import (
    "flag"
    "fmt"
    "github.com/golang/glog"
    "html/template"
    "net/http"
    "os"
    "strconv"
)

type config struct {
    Risky    bool
    Port     int
    Greeting string
}

type server struct {
    C    *config
    Path string
    Version int
}

const (
    version = 0
    tmplName = "whatever"
    tmplBody = `
{{define "` + tmplName + `" -}}
<html><body>
Version {{.Version}} : {{if .C.Risky}}<em>{{end}}
{{- .C.Greeting}}{{if .C.Risky}}</em>{{end}} {{.Path}}
</body></html>
{{end}}
`)

var (
    c                  *config
    enableRiskyFeature = flag.Bool("enableRiskyFeature", false,
        "Enables some risky feature.")
    port = flag.Int("port", 8080, "Port at which HTTP is served.")
    tmpl = template.Must(template.New("main").Parse(tmplBody))
)

func getConfig() *config {
    flag.Parse()
    greeting := os.Getenv("TUTORIAL_GREETING")
    if len(greeting) == 0 {
        greeting = "Hello"
    }
    return &config{*enableRiskyFeature, *port, greeting}
}

func handler(w http.ResponseWriter, r *http.Request) {
    path := r.URL.Path[1:]
    if err := tmpl.ExecuteTemplate(
        w, tmplName, &server{c, path, version}); err != nil {
        glog.Fatal(err)
    }
    if path == "quit" {
        fmt.Println("All done.")
        os.Exit(0)
    }
}

func main() {
    c = getConfig()
    http.HandleFunc("/", handler)
    hostPort := ":" + strconv.Itoa(c.Port)
    fmt.Println("Serving at " + hostPort)
    if err := http.ListenAndServe(hostPort, nil); err != nil {
        glog.Fatal(err)
    }
}
EOF
```

Build version 1 of the program to see that it works.

<!-- @defineFunctionToBuild -->
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

<!-- @buildBinaryAtVersion1 -->
```
rm -f $TUT_IMG_PATH
tut_BuildProgram $TUT_IMG_V1
file $TUT_IMG_PATH
ls -sh $TUT_IMG_PATH
```

<!-- @defineFunctionToRunAndKillLocalServer -->
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

<!-- @runAndKillLocalServer -->
```
TUTORIAL_GREETING=salutations ${TUT_IMG_PATH} \
    --enableRiskyFeature --port 8100 &
tut_RequestAndQuit 8100 godzilla
```
