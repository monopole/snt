# Testing

[mdrip]: https://github.com/monopole/mdrip

Download the content and [mdrip]:

```
dir=$(mktemp -d)
repo=https://github.com/monopole/snt.git
git clone $repo $dir/tutorial
mkdir -p $dir/bin
GOBIN=$dir/bin go install github.com/monopole/mdrip
```

Test the tutorial:

```
$dir/bin/mdrip --mode test --label test \
  --blockTimeOut 15m $dir/tutorial
echo $?
```
If the above exits with $?==0, everything with the @test label works.

The long timeout allows ISO downloads over dodgy wifi.

## More verbose testing

A more verbose, edittable form of the above:

```
cd $dir/tutorial
$dir/bin/mdrip \
    --mode test \
    --label test \
    --blockTimeOut 15m \
    --alsologtostderr \
    --v 2 \
    --stderrthreshold INFO \
    ./README.md \
    ./startCluster/minikube.md \
    ./startCluster/confirm.md \
    ./buildAServer.md \
    ./containerize.md \
    ./k8sReview/namespaces.md \
    ./k8sReview/pods.md \
    ./k8sReview/services.md \
    ./k8sReview/replication.md \
    ./k8sReview/deployment.md \
    ./k8sReview/configMap.md
```
etc. etc.

## Fast forward your environment

Runs blocks your current shell,
leaving the modified environment to play with.

```
cd $dir/tutorial
eval "$($dir/bin/mdrip --label test \
  ./README.md \
  ./startCluster/minikube.md \
  ./startCluster/confirm.md \
  ./buildAServer.md \
  ./containerize.md \
  ./k8sReview/namespaces.md \
  ./k8sReview/pods.md \
)"
```

## Re-establish environment

Assumes that all downloads are done and that the cluster is up.

Just run the blocks labelled @env to establish the bash environment.

```
eval "$($dir/bin/mdrip --label env $dir/tutorial)"
```

Do the following to export all functions,
to allow partial tests in subprocesses to work.

```
while IFS= read -r line; do export -f $line; done < <(
    declare -F | grep tut_ | awk '{ print $3 }')
```

E.g., after the above, try running just
the service tutorial in test mode.

```
cd $dir/tutorial
$dir/bin/mdrip \
    --mode test \
    --label test \
    --blockTimeOut 2m \
    --alsologtostderr \
    --v 2 \
    --stderrthreshold INFO \
    ./k8sReview/services.md
```
