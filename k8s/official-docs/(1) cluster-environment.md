## local test environment:
- kubectl intalled 
- minikube installed 
- a container runtime (e.g. docker) installed as minikube driver to impelment a one node cluster as container

## production:
consider:
- **Availability** 
    1. sprating worker nodes to at least two nodes 
    2. HA and load balancinfg between two api servers 
    3. replicating control plane components to multiple nodes
- **scale**: increase and reduce resources on demand
- **security and access management**: who can access the cluster resources

### Production Control Plane
to keep the cluster up and running and ensuring that it can be repaired if something goes wrong:
- **user deployment tool**: kubeadm
- **certificates**: secure communications between control plane services
- **Configure load balancer for apiserver**
- **Separate and backup etcd service**
- **Create multiple control plane systems**
- **Span multiple zones**: spreading a cluster across multiple zones
- **Manage on-going features**: like certificate managment or upgrading kubeadm cluster

### Production worker nodes
consider how you want to manage your worker nodes:
- **Configure nodes**: could be a physical ro virtual machine as node. install and run the appropriate node services. consider:
    * memory, CPU, disk speed, storage
    * GPU processors or VM isolation if needed
- **Validate nodes**: ensure that node meets the requiremnets to joim the cluster
- **scale nodes**: total capacity of your cluster 
- **autoscale nodes**: tools available to automatically manage your nodes
- **Set up node health checks**: e.g. Node Problem Detector daemon

### Production user management
In production, you will want more accounts to work the cluster with different levels of access to different namespaces. you need to consider authentication(identity) and authorization(permission to do task):
- **Authentication**
- **Authorization**
- **Set the authorization mode**
- **Create user certificates and role bindings (RBAC)**
- **Consider Admission Controllers**

### Set limits on workload resources
Demands from production workloads can cause pressure both inside and outside of the Kubernetes control plane. Consider these items when setting up for the needs of your cluster's workloads:
- **Set namespace limits**
- **Prepare for DNS demand**
- **Create additional service accounts**

>[source]
> https://kubernetes.io/docs/setup/production-environment