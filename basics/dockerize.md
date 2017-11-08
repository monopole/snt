# Make docker images

Kubernetes wants pods to pull their container images
from a (presumably remote and trustworthy) server
called a container registry, so that's a first step


If you're using minikube for your cluster, you'll use a
registry built into minikube.  This is done via:

 1. Configuring your `docker` command to talk to the
    docker deamon in the running minikube, such that
    when you enter `docker push` you'll push directly
    into minikube's docker storage, and pods will pull
    from its cache.

 2. Creating pods with `imagePullPolicy: IfNotPresent`

<!-- @useThisIfUsingMinikube -->
```
TUT_CON_HOST=""
```


[Google container registry]: http://gcr.io
If you're using GKE, you'll use the
[Google container registry].

<!-- @useThisIfUsingGKE -->
```
TUT_CON_HOST="gcr.io"
```

Define an image tag to use as an argument to various
docker commands and as the value of the `image` field
in kubernetes pod definitions.

<!-- @defineImageTag -->
```
if [ -n "$TUT_CON_HOST" ]; then
  # use remote registry
  TUT_IMG_TAG=$TUT_CON_HOST/$TUT_PROJECT_ID/$TUT_IMG_NAME
else
  # use local registry
  TUT_IMG_TAG=$TUT_PROJECT_ID/$TUT_IMG_NAME
  eval $(minikube docker-env)
fi
echo "TUT_IMG_TAG=$TUT_IMG_TAG"
echo "DOCKER_HOST=$DOCKER_HOST"
```


> _TODO_: make a better canonical test to distinguish minikube from GKE use.

> _TODO_: maybe use non-VM but local registry via `minikube start --insecure-registry`

## Create images

Put the web server into a container image.

<!-- @removeAllLocalDockerImages -->
```
# docker rm $(docker stop $(docker ps -aq))
docker rmi $TUT_IMG_TAG:$TUT_IMG_V1
docker rmi $TUT_IMG_TAG:$TUT_IMG_V2
```

<!-- @peekAtCurrentlyRunningContainers -->
```
# If running a local minikube, there will be many processes.
# If running on GKE, there might not be anything here.
docker ps -a
```

<!-- @defineFunctionToCreateDockerImage -->
```
function tut_BuildDockerImage {
  local tag=$TUT_IMG_TAG:$1  # Add version to tag
  local dockerFile=$TUT_DIR/src/Dockerfile
  cat <<EOF >$dockerFile
FROM scratch
ADD ${TUT_IMG_NAME} /
CMD ["/${TUT_IMG_NAME}"]
EOF
  docker build -t $tag -f $dockerFile $TUT_DIR/src
  rm $dockerFile
}
```

<!-- @createDockerImageVersion1 -->
```
tut_BuildDockerImage $TUT_IMG_V1
```

<!-- @listRelevantImages -->
```
docker images --no-trunc | grep $TUT_IMG_NAME
```

Run the image locally to test it:

<!-- @runDockerImage -->
```
docker run -d -p 8080:8080 $TUT_IMG_TAG:$TUT_IMG_V1
docker ps | grep $TUT_IMG_TAG

# Handy trick:
#   docker ps -a # get container ID
#   docker exec -it {containerId} bash
# then cd /tmp to examine logs

if [ -n "$TUT_CON_HOST" ]; then
  host=localhost
else
  host=$(minikube ip)
fi

curl -m 1 $host:8080/kingGhidorah
curl -m 1 $host:8080/quit
```

<!-- @confirmTheServerIsGone -->
```
docker ps | grep $TUT_IMG_TAG
```

Build another image at version 2 to allow
rollout/rollback practice later:

<!-- @buildVersion2 -->
```
tut_BuildProgram     $TUT_IMG_V2
tut_BuildDockerImage $TUT_IMG_V2
```

<!-- @confirmLocalDockerCache -->
```
ls -1sh $TUT_DIR/src
docker images | grep ${TUT_IMG_NAME}
```

[GCR]: http://gcr.io

## Push images to [GCR]

This section only applies if using [GCR]
instead of minikube's local docker.

<!-- @pushToGcr -->
```
if [ -n "$TUT_CON_HOST" ]; then
  docker push $TUT_IMG_TAG:$TUT_IMG_V1
  docker push $TUT_IMG_TAG:$TUT_IMG_V2
else
 # We've already built the images into minikube's
 # docker's cache.
fi
```

Having done push, make sure pull works.

<!-- @exerciseGcr -->
```
if [ -n "$TUT_CON_HOST" ]; then
  # Confirm the cached image
  docker images | grep $TUT_IMG_TAG
  # Remove the cached image
  docker rmi $TUT_IMG_TAG:$TUT_IMG_V2
  # Confirm it's gone
  docker images | grep $TUT_IMG_TAG
  # Pull it from the registry
  docker pull $TUT_IMG_TAG:$TUT_IMG_V2
  # Confirm it's back.
  docker images | grep $TUT_IMG_TAG
fi
```

Optionally start by deleting old images (if any):

<!-- @deleteImages -->
```
if [ -n "$TUT_CON_HOST" ]; then
  gcloud --quiet container images delete $TUT_IMG_TAG:$TUT_IMG_V1
  gcloud --quiet container images delete $TUT_IMG_TAG:$TUT_IMG_V2
fi
```

See also these [container deletion instructions].

[container deletion instructions]: https://cloud.google.com/container-registry/docs/quickstart


Then upload:

<!-- @uploadImages -->
```
if [ -n "$TUT_CON_HOST" ]; then
  gcloud docker -- push $TUT_IMG_TAG:$TUT_IMG_V1
  gcloud docker -- push $TUT_IMG_TAG:$TUT_IMG_V2
fi
```


List the cloud-homed images:

<!-- @listImages -->
```
if [ -n "$TUT_CON_HOST" ]; then
  (
  gcloud container images list --repository $TUT_CON_HOST/$TUT_PROJECT_ID
  echo "--------------------"
  gcloud container images list-tags $TUT_IMG_TAG
  echo "--------------------"
  gcloud container images list-tags --format='get(digest)' $TUT_IMG_TAG
  )
fi
```

## Cleanup

The container images and source code in `$TUT_DIR` are no longer needed:

<!-- @lsTutDir -->
```
ls -C1 $TUT_DIR/src/${TUT_IMG_NAME}*
rm $TUT_DIR/src/${TUT_IMG_NAME}*
```


   <!--
   notes about using a local, but not inside minikube, docker daemon:

   Flag `-p` publishes the container port (5000 in this case) to the host.
   Optionally add flag `--restart always` if it crashes for some reason.
   The `--name` flag assignes the name, and `registry:2` is the
   [container's tag](https://hub.docker.com/_/registry/).

   docker run -d -p 5000:5000 --name registry registry:2
   # Stop it with: docker stop registry
   -->
