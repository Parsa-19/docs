```
# deploy by the official nginx image
kubectl create deployment nginx --image=nginx

# confirm on which node the pod has been created (FIND THE IP ADDRESS)
kubectl get pod -o wide

# expose the nginx pod through a service
kubectl expose deployment nginx --port=80 --type=NodePort

# find the nodeort you created from the service. e.g. 80:32202 (FIND OUT PORT)
kubectl get svc

# access the nginx page on browser on (NODE_IP_ADDRESS:PORT)

```