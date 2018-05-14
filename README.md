# sre-test
Deployment configurations for sre test.

```
gcloud container clusters create sample --num-nodes=3 --scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"
gcloud container clusters get-credentials sample
kubectl create ns jenkins
gcloud compute images create jenkins-home-image --source-uri https://storage.googleapis.com/solutions-public-assets/jenkins-cd/jenkins-home-v3.tar.gz
gcloud compute disks create jenkins-home --image jenkins-home-image --zone europe-west1-b
kubectl create secret generic jenkins --from-file=jenkins/k8s/options --namespace=jenkins
kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value account)
kubectl apply -f jenkins/k8s/

# ONLY FOR TEST PURPOSES, REAL DOMAIN CERTIFICATE NEEDED
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /tmp/tls.key -out /tmp/tls.crt -subj "/CN=jenkins/O=jenkins"
kubectl create secret generic tls --from-file=/tmp/tls.crt --from-file=/tmp/tls.key --namespace jenkins

kubectl apply -f jenkins/k8s/lb/ingress.yaml
kubectl describe ingress jenkins --namespace jenkins
```
