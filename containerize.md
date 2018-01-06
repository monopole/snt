# Containerize the Server

> _Learn to put a Go server into a container._
>
> _Time: 3m_

k8s exclusively runs containers (tar balls with metadata),
pulling them as needed from a container registry.

The following puts `tuthello` into a container, and
in turn puts the container into a registry.

 * If you're using minikube for your cluster, you'll use a
   registry built into minikube.
   1. You'll configue your `docker` command to talk to the
      docker deamon in the running minikube, such that
      when you enter `docker push` you'll push directly
      into minikube's docker storage.
   2. You'll create pods with `imagePullPolicy: IfNotPresent`

 * If you're using GKE, you'll use the
   Google container registry at http://gcr.io.

## Set up environment

Define an image tag to use as an argument to various
docker commands and as the value of the `image` field
in kubernetes pod definitions.

<!-- @defineImageTag @env @test -->
```
export TUT_IMG_TAG=tuthello
if tut_isMinikube; then
  # use local registry
  eval $($MINIKUBE_HOME/minikube docker-env)
  echo "DOCKER_HOST=$DOCKER_HOST"
else
  # Use GCR, which requires a longer tag.
  TUT_IMG_TAG=gcr.io/$TUT_PROJECT_ID/tuthello
fi
echo "TUT_IMG_TAG=$TUT_IMG_TAG"
```

> _TODO_: maybe use non-VM but local registry via `minikube start --insecure-registry`

## Create images

Harmlessly assure there are no images left over from
a previous pass through these commands.

<!-- @rmDockerImages -->
```
# docker rm $(docker stop $(docker ps -aq))
docker rmi $TUT_IMG_TAG:1
docker rmi $TUT_IMG_TAG:2
```

See what processes are running in the container.

If running a local minikube, there will be many processes.
If running on GKE, there might not be anything here.

<!-- @peekAtRunning @test -->
```
docker ps -a
```

<!-- @funcCreateImage @env @test -->
```
function tut_buildDockerImage {
  local tag=$TUT_IMG_TAG:$1  # Add version to tag
  local dockerFile=$TUT_DIR/src/Dockerfile
  cat <<EOF >$dockerFile
FROM scratch
ADD tuthello /
CMD ["/tuthello"]
EOF
  docker build -t $tag -f $dockerFile $TUT_DIR/src
  rm $dockerFile
}
```

<!-- @createImageV1 @test -->
```
tut_buildDockerImage 1
```

<!-- @listImages @test -->
```
docker images --no-trunc | grep tuthello
```

Sanity check the container image by running it:

<!-- @runDockerImage @test -->
```
docker run -d -p 8080:8080 $TUT_IMG_TAG:1
docker ps | grep $TUT_IMG_TAG

if tut_isMinikube; then
  host=$($MINIKUBE_HOME/minikube ip)
else
  host=localhost
fi

curl --fail --silent -m 1 $host:8080/kingGhidorah
curl --fail --silent -m 1 $host:8080/quit
```

> To examine logs inside docker:
>
> ```
>  docker ps -a # get container ID
>  docker exec -it {containerId} bash
> ```
>
> then `cd /tmp` to examine logs.

Build another image at version 2 to allow
rollout/rollback practice later:

<!-- @buildVersion2 @test -->
```
tut_buildProgram     2
tut_buildDockerImage 2
```

<!-- @confirmDockerCache @test -->
```
docker images | grep tuthello
```

[GCR]: http://gcr.io

## Push images to [GCR] if using GKE.

This section only applies if using GKE instead of minikube.

Optionally start by deleting old images (if any):

<!-- @deleteImages -->
```
if ! tut_isMinikube; then
  gcloud --quiet container images delete $TUT_IMG_TAG:1
  gcloud --quiet container images delete $TUT_IMG_TAG:2
fi
```

See also these [container deletion instructions].

[container deletion instructions]: https://cloud.google.com/container-registry/docs/quickstart

Then upload:

<!-- @uploadImages -->
```
if ! tut_isMinikube; then
  gcloud docker -- push $TUT_IMG_TAG:1
  gcloud docker -- push $TUT_IMG_TAG:2
fi
```

List the cloud-homed images:

<!-- @listImages -->
```
if ! tut_isMinikube; then
  (
  gcloud container images list \
    --repository gcr.io/$TUT_PROJECT_ID
  echo "--------------------"
  gcloud container images list-tags $TUT_IMG_TAG
  echo "--------------------"
  gcloud container images list-tags \
    --format='get(digest)' $TUT_IMG_TAG
  )
fi
```

## Cleanup

The container images and source code in `$TUT_DIR` are no longer needed:

<!-- @lsSrc @test -->
```
ls -C1 $TUT_DIR/src
```

<!-- @removeSrc @test -->
```
rm -rf $TUT_DIR/src
ls $TUT_DIR
```
