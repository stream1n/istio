Steps from https://cloud.google.com/solutions/authenticating-cloud-run-on-gke-end-users-using-istio-and-identity-platform


ZONE=us-central1-c
CLUSTER=cloud-run-gke-auth-tutorial

gcloud config set compute/zone $ZONE
gcloud config set run/cluster $CLUSTER
gcloud config set run/cluster_location $ZONE

export GOOGLE_CLOUD_PROJECT=streamin-263110

Create cluster remembering to enable datastore access

gcloud beta container clusters create $CLUSTER --addons HorizontalPodAutoscaling,HttpLoadBalancing,CloudRun --enable-ip-alias --enable-stackdriver-kubernetes --machine-type n1-standard-2 --scopes=datastore,gke-default

Check for IP
kubectl get services istio-ingress -n gke-system --watch

export EXTERNAL_IP=$(kubectl get services istio-ingress \
    --namespace gke-system \
    --output jsonpath='{.status.loadBalancer.ingress[0].ip}')

echo $EXTERNAL_IP and add it to the allowed Identity Platform -> Settings -> Security

kubectl create namespace public
kubectl create namespace api

Then run the cloud builds to deploy services to these namespaces

Then create the istio rules

kubectl apply -f istio/virtualservice.yaml

envsubst < istio/authenticationpolicy.template.yaml | kubectl apply -f -

Should work then
