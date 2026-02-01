# run the project

> [NOTICE]
> source: https://www.youtube.com/watch?v=s_o8dwzRlu4

- put this projects root dir in your linux vm.
- adjust the vm to be brigded on network.
- stop the firewall or add its rule on the ports.
- you need to run theses commands in order to be able access the application through your windows host browser
```
minikube start
kubectl apply -f mongo-config.yaml
kubectl apply -f mongo-secret.yaml
kubectl apply -f mongo.yaml
kubectl apply -f webapp.yaml
kubectl get all
kubectl port-forward pod/WEBAPP_POD_NAME 8080:CONTAINER_PORT --address 0.0.0.0

# now you can access the app on VM IP on port 8080 on windows
http://VM_IP:8080

# you can test the applicaton with minikube node ip with the nodePort configured inside the VM (30100 is the nodePort configured in this porject)
minikube ip
curl MINIKUBE_NODE_IP:30100
```
webapp CONTAINER_PORT(inside the pod) = 3000<br>
webapp SERVICE_TARGET_PORT(point to container port) = 3000<br>
webapp SERVCIE_PORT(could be anything) = 3000<br>
webapp nodePort(port assigned to the node with range 30000/32767) = 30100<br>

mongo CONTAINER_PORT(inside the pod) = 27017<br>
mongo SERVICE_TARGET_PORT(point to container port) = 27017<br>
mongo SERVCIE_PORT(could be anything) = 27017<br>
