
Network Policy on Google Kubernetes Engine

gcloud auth list

gcloud config list project

Clone the resources needed for this lab by running:

git clone https://github.com/GoogleCloudPlatform/gke-network-policy-demo.git
cd gke-network-policy-demo

Run the following to set a region and zone

gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a

make setup-project

cat terraform/terraform.tfvars

Set terraform to the latest version:

sed -i 's/~> 2.10.0/~> 2.14.0/g' terraform/provider.tf

Provisioning the Kubernetes Engine Cluster
make tf-apply

gcloud container clusters describe gke-demo-cluster | grep  -A2 networkPolicy

gcloud compute ssh gke-demo-bastion

The test application consists of one simple HTTP server, deployed as hello-server, and two clients, one of which will be labeled app=hello and the other app=not-hello.

kubectl apply -f ./manifests/hello-app/

kubectl get pods

Confirm default access to hello server

kubectl logs --tail 10 -f $(kubectl get pods -oname -l app=hello)

Check the blocker client logs

kubectl logs --tail 10 -f $(kubectl get pods -oname -l app=not-hello)

Now you will block access to the hello-server pod from all pods that are not labeled with app=hello

kubectl apply -f ./manifests/network-policy.yaml

kubectl logs --tail 10 -f $(kubectl get pods -oname -l app=not-hello)


kubectl delete -f ./manifests/network-policy.yaml

Policy based on namespace:

kubectl create -f ./manifests/network-policy-namespaced.yaml

kubectl logs --tail 10 -f $(kubectl get pods -oname -l app=hello)

kubectl logs --tail 10 -f -n hello-apps $(kubectl get pods -oname -l app=hello -n hello-apps)
