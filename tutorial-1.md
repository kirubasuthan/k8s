# Create a Nginx Pod, Serice and Port forward to locahost port 

# Ensure Docker Desktop is installed and Kubernetes is gnabled 
kubectl get nodes

# Create Nginx Pod
kubectl run nginx --image nginx --labels="run=nginx"

# See events
kubectl get events 

# Connect to nginx pod
kubectl exec -it nginx -- bash

# Display Resource (CPU/Mem usage)
kubectl top nodes
kubectl top pods 

# Create Node Port Service
kubectl expose pod nginx --port=88 --target-port=80 --name=nginx-service
kubectl get pod,svc -l run=nginx

kubectl port-forward svc/nginx-service 88:88

# Delete pod and svc

kubectl delete pods/nginx
kubectl delete svc/nginx-service
