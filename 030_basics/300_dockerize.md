# Make docker images

Kubernetes is built to allow pods to pull their
container images from a (presumably remote and
trustworthy) server called a container registry.

[Google container registry]: http://gcr.io

Below make a choice between using a local registry (to
avoid testing complexity), and using the [Google
container registry] (you'd want to use an independent,
dedicated service in real life).

minikube runs a docker daemon in its VM to maintain
images it needs.  When the kubernetes "cluster" running
inside the minikube VM attempts to pull an image to run
a pod, it will consult the docker cache in said VM.

To remain completely local:

 1. Configure your local docker client (the one you use
    when you run docker at the command line) to talk
    to the docker deamon in the running minikube.
    Then, when you enter `docker push` you'll push directly
    into minikube's docker storage.

 2. create pods with `imagePullPolicy: IfNotPresent`


<!-- @defineRegistryContainerHostEnvVar -->
```
# Pick one
TUT_CON_HOST="gcr.io"
TUT_CON_HOST=""
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

<!-- maybe use non-VM but local registry via `minikube start --insecure-registry` -->

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

curl -m 1 $(minikube ip):8080/kingGhidorah
curl -m 1 $(minikube ip):8080/quit

# Confirm gone
docker ps | grep $TUT_IMG_TAG
```

Build another image at version 2
so we can practice rollouts later:

<!-- @buildVersion2 -->
```
tut_BuildProgram     $TUT_IMG_V2
tut_BuildDockerImage $TUT_IMG_V2
```

<!-- @confirmLocalDockerCache -->
```
ls -1sh $TUT_DIR
ls -1sh $TUT_DIR/src
docker images | grep ${TUT_IMG_NAME}
```


## Push the images to the container registry


### Using a local registry

<!--


Flag `-p` publishes the container port (5000 in this case) to the host.
Optionally add flag `--restart always` if it crashes for some reason.
The `--name` flag assignes the name, and `registry:2` is the
[container's tag](https://hub.docker.com/_/registry/).


```
docker run -d -p 5000:5000 --name registry registry:2
# Stop it with: docker stop registry
```
-->


__TODO: remove this:__

In this case we've built the containers right in the
daemon that plans to use them. If we were using a
remote registry, we'd push to it now:

> ```
> docker push $TUT_IMG_TAG:$TUT_IMG_V1
> docker push $TUT_IMG_TAG:$TUT_IMG_V2
> ```
>
> For fun, list it an image, remove it from the local cache, see
> its gone from the list, pull it, see it return to the
> list:
>
> ```
> docker images | grep $TUT_IMG_TAG
> docker rmi $TUT_IMG_TAG:$TUT_IMG_V2
> docker images | grep $TUT_IMG_TAG
> docker pull $TUT_IMG_TAG:$TUT_IMG_V2
> docker images | grep $TUT_IMG_TAG
> ```




### Using Google container registry

Perhaps start by deleting old images:

<!-- @deleteImages -->
```
gcloud --quiet container images delete $TUT_IMG_TAG:$TUT_IMG_V1
gcloud --quiet container images delete $TUT_IMG_TAG:$TUT_IMG_V2
```

See also these [container deletion instructions].

[container deletion instructions]: https://cloud.google.com/container-registry/docs/quickstart


Then upload:

<!-- @uploadImages -->
```
gcloud docker -- push $TUT_IMG_TAG:$TUT_IMG_V1
gcloud docker -- push $TUT_IMG_TAG:$TUT_IMG_V2
```


List the cloud-homed images:

<!-- @listImages -->
```
(
gcloud container images list --repository $TUT_CON_HOST/$TUT_PROJECT_ID
echo "--------------------"
gcloud container images list-tags $TUT_IMG_TAG
echo "--------------------"
gcloud container images list-tags --format='get(digest)' $TUT_IMG_TAG
)
```

## Cleanup

The container images and source code in `$TUT_DIR` are no longer needed:

<!-- @lsTutDir -->
```
ls -C1 $TUT_DIR/src/${TUT_IMG_NAME}*
rm $TUT_DIR/src/${TUT_IMG_NAME}*
```
