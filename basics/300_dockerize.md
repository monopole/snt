# Make docker images

Web server in hand, dockerize it.

<!-- @removeAllLocalDockerImages -->
```
docker rm $(docker stop $(docker ps -aq))
docker rmi $TUT_IMG_TAG:$TUT_IMG_V1
docker rmi $TUT_IMG_TAG:$TUT_IMG_V2
```

<!-- @assureNoContainersRunningLocally -->
```
docker ps -a
```

<!-- @defineFunctionToCreateDockerImage -->
```
function tut_BuildDockerImage {
  local tag=$TUT_IMG_TAG:$1  # Add version to tag
  local dockerFile=$TUT_DIR/Dockerfile
  cat <<EOF >$dockerFile
FROM scratch
ADD ${TUT_IMG_NAME} /
CMD ["/${TUT_IMG_NAME}"]
EOF
  docker build -t $tag -f $dockerFile $TUT_DIR
  rm $dockerFile
}
```

TODO: convert to https, install certs/secrets

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
docker ps # Should show our server

# Handy trick:
#   docker ps -a # get container ID
#   docker exec -it {containerId} bash
# then cd /tmp to examine logs

tut_RequestAndQuit 8080 kingGhidorah
sleep 1
docker ps # Should show no processes running
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
docker images | grep ${TUT_IMG_NAME}
```


### Upload docker images

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

The files in `$TUT_DIR` are no longer needed:

<!-- @lsTutDir -->
```
ls -sh $TUT_DIR
```

<!-- @cleanTutDir -->
```
rm $TUT_DIR/*
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
