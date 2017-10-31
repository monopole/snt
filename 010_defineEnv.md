## Define an Environment

Any files created by commands that follow are placed in a
disposable directory:

<!-- @makeTutorialWorkingDirectory-->
```
TUT_DIR=$(mktemp -d)
```

Environment variables used to assure
consistency across command blocks:

<!-- @setInitialGlobalEnvVars -->
```
# Various k8s things to be made
TUT_NAMESPACE_NAME=ns-beansprout
TUT_INGRESS_NAME=ing-broccoli
TUT_SERVICE_NAME=svc-eggplant
TUT_POD_NAME=pod-tomato
TUT_REPSET_NAME=rep-asparagus
TUT_DEPLOY_NAME=dep-kale
TUT_CFG_NAME_1=cfg-parsley
TUT_CFG_NAME_2=cfg-cilantro
TUT_STATIC_IP_NAME=peach-static-ip

# Container stuff
TUT_CON_CPU=100m     # 10% of a CPU
TUT_CON_MEMORY=100Mi
TUT_CON_NAME=cnt-carrot
TUT_CON_PORT_NAME=port-pumpkin
# Match the binary's default value - till the command line example.
TUT_CON_PORT_VALUE=8080

# Image stuff
TUT_IMG_NAME=radishwine
TUT_IMG_PATH=$TUT_DIR/$TUT_IMG_NAME
TUT_IMG_V1=1
TUT_IMG_V2=2

# Label to connect load balancer to container
TUT_APP_LABEL=penguin-toast

# Cluster load balancer port
TUT_EXT_PORT=8088

# Where to host containers
TUT_CON_HOST=gcr.io

# Arbitrary name for the cluster made below.
TUT_CLUSTER_NAME=cluster-spinach
```
