# Testing

> Test or rerun all or part of the tutorial
>
> Time: 1-5min

## Requirements

[mdrip]: https://github.com/monopole/mdrip

Download the content and [mdrip]:

```
dir=$(mktemp -d)
repo=https://github.com/monopole/snt.git
git clone $repo $dir/tutorial
mkdir -p $dir/bin
GOBIN=$dir/bin go install github.com/monopole/mdrip
PATH=$dir/bin:$PATH
```

## Test the entire tutorial

```
mdrip --mode test --label test \
  --blockTimeOut 15m $dir/tutorial
echo $?
```
If the above exits with $?==0, everything with the @test label works.

The long timeout allows ISO downloads over dodgy wifi.

## Verbose, custom testing

A more verbose, edittable form of the above:

```
cd $dir/tutorial
```

```
mdrip \
  --mode test \
  --label test \
  --blockTimeOut 15m \
  --alsologtostderr \
  --v 2 \
  --stderrthreshold INFO \
  ./README.md \
  ./environment.md \
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
```

```
eval "$(mdrip --label test \
  ./README.md \
  ./environment.md \
  ./startCluster/minikube.md \
  ./startCluster/confirm.md \
  ./buildAServer.md \
  ./containerize.md \
  ./k8sReview/namespaces.md \
  ./k8sReview/pods.md \
)"
```

## Re-establish environment

Suppose you accidentally close your working shell,
after you did all the downloads and started a cluster.

To re-establish the bash env in a new shell, run this:

```
eval "$(mdrip --label env $dir/tutorial)"
```

## Partial tests

To run some part of the tutorial in test mode
(primarily as a means to debug that section) without
rerunning everything (e.g. to run against an existing
cluster), the bash functions in the environment must be
exported to make them available to the test
subprocess.

Do this:
```
while IFS= read -r line; do export -f $line; done < <(
    declare -F | grep tut_ | awk '{ print $3 }')
```

E.g., after the above, try running just
the service tutorial in test mode.

```
cd $dir/tutorial
```

```
mdrip \
  --mode test \
  --label test \
  --blockTimeOut 2m \
  --alsologtostderr \
  --v 2 \
  --stderrthreshold INFO \
  ./k8sReview/services.md
```
