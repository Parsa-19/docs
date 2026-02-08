# goal
to bring up a wordpress container and connect it to mysql container (deployed in k8s).
# how to run
### in a ubuntu vm
```
kubectl apply -k ./
kubectl get pod # to get the name of wordpress pod
kubectl port-forward pod/[WORDPRESS_POD] 8080:80 --address 0.0.0.0
```
### in your windows host
open the browser and type: VM_IP:8080