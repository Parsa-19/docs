```
# inside your vm where the docker and minikube and kubectl installed:

minikube start
minikube status

# on another shell
minikube dashboard

# on another shell
kubectl proxy --address='0.0.0.0' --accept-hosts='.*'

# go to your Windows (or any system that you had run the vm on before) browser and paste this and also replace the VM_IP
http://VM_IP:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/workloads?namespace=_all 
```