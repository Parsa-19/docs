```
kubectl apply -f [YAML_CONFIG_FILE]
kubectl delete -f [DEPLOYMENT_NAME]

kubectl get all
kubectl get pod
kubectl get deployment
kubectl get service
kubectl get configmap
kubectl get secret
kubectl get node
kubectl get node -o wide
kubectl get pod -o wide # get pod's ip addresses

kubectl create deployment [DEPLOYMENT_NAME] [IMAGE:version]
kubectl delete deployment [DEPLOYMENT_NAME]

kubectl describe service [SERVICE_NAME] # see which pod/pods the service is pointing to
kubectl describe pod [POD_NAME]

kubectl edit pod [POD_NAME]
kubectl logs [POD_NAME] -f
kubectl exec -it [POD_NAME] -- bin/bash

kuebctl get deployment [DEPLOYMENT_NAME] -o yaml # get the config file for specific deployment
```
