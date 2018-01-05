# Trouble shooting

<!-- @connectToControlPlane -->
```
gcloud container clusters get-credentials $TUT_CLUSTER_NAME \
  --zone us-west1-a --project $TUT_PROJECT_ID
kubectl proxy
# then open browser to http://localhost:8001/ui
```
