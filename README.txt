# Drupal container

# Tutorials: 
# https://cloud.google.com/container-engine/docs/quickstart
# adapted from https://cloud.google.com/container-engine/docs/tutorials/persistent-disk/#wordpress_pod

# Follow https://developers.google.com/identity/protocols/application-default-credentials to get credentials and store the json file in this folder

# Set environment variable to this set of creds
export GOOGLE_APPLICATION_CREDENTIALS=DrupalContainerEngine-f8ef8a6dc869.json

# Update gcloud
gcloud components update

# install kubernetes command line
gcloud components install kubectl

# Set to europe for all future commands
gcloud config set compute/zone europe-west1-b

# List properties
gcloud config list

# Set to our project. Make sure this exists in the UI.
gcloud config set project drupalcontainerengine

# initialize the cluster
gcloud container clusters create drupalpd \
--machine-type "f1-micro" \
--image-type "GCI" \
--disk-size "100" \
--scopes "https://www.googleapis.com/auth/compute","https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --num-nodes 2 \
--network "default" \
--enable-cloud-logging \
--no-enable-cloud-monitoring \
--enable-autoupgrade

# Log in to our kubernetes panel
gcloud container clusters get-credentials drupalpd \
    --zone europe-west1-b --project drupalcontainerengine

kubectl proxy
# Browse to http://localhost:8001/ui

# mysql
# Set up a managed mysql instance and import a database into it. Next, copy the nickveenhofbe folder and modify the information in pods.yaml to match your setup.

# Create NFS Server PV & Claim
kubectl create -f nfs-shared1-pv.yaml

# Create NFS Server
kubectl create -f nfs-shared1-server.yaml

# Create a folder for your site
kubectl exec -it nfs-shared1-server-rxlfk mkdir /exports/nickveenhofbe

# Create NFS Service
kubectl create -f nfs-shared1-service.yaml

# Get the IP Address for our NFS Shared1 server
kubectl describe service nfs-shared1-server

# Replace IP address in nickveenhofbe/nfs-pv.yaml
# And Create NFS folder volume to mount for our site
kubectl create -f nickveenhofbe/nfs-pv.yaml

# Claim this NFS folder volume
kubectl create -f nickveenhofbe/nfs-pv-claim.yaml

# Start the pods (2 of them)
kubectl create -f nickveenhofbe/pods.yaml

# Update the Drupal pod
kubectl replace -f nickveenhofbe/pods.yaml

# Create drupal service
kubectl create -f nickveenhofbe/lb.yaml

# verify that it points to defined endpoints (eg, ip's)
kubectl describe service nickveenhofbe

# Get our properties in order to visit our site. Wait till it mentions a property like: "LoadBalancer Ingress". It might take a while before the load balancer is created.
kubectl describe service nickveenhofbe

# Visit the site on the Public IP that is found in the load balancer
# and behold the beauty of your own HA site.

# Caveats & TODOs
# It does not properly use the NFS voluem yet to share files for sites/default/files so file uploads will only be seen in 1 of the 2 pods.