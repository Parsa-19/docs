## my general idea of the whole thing
what I went through when decided I need to know how to create cluster:
### this example scenario:
create a k8s cluster between three ubuntu nodes using kubeadm. <br>
network configuration is: <br> 
two network adapters for each vm (I am implementing this in virtual box) one adapter is NAT (10.0.2.0/24) and one is hostonly (192.168.55.0/24).



# 1. network configuratoin

#### load br_netfilter module
```
$ sudo modprobe br_netfilter

$ lsmod | grep br_netfilter

$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF
```

apply immediately without reboot:
```
sudo sysctl --system
```

verify:
```
sysctl net.ipv4.ip_forward
```

make the module persistent:
```
echo br_netfilter | sudo tee /etc/modules-load.d/k8s.conf
```




# 2. cgroups configuration
cgroups are used to constrain resources that are allocated to processes. Both kubelet and container runtime needs to access system cgroups to enforce resource management for pods and containers. They also have to use the same cgroup driver to work properly.<br>
there are two cgroup drivers:
- **cgroupfs** = the default for kubelet but it is not recommended when systemd is the init system. 
- **systemd** = in systemd the init process generates and consumes the root cgroup and act as cgroup manager.
<br>
you need to make sure that both kubelet and container runtimes are using the same cgroup driver.<br>
the cgroup setup is explained in later topics.



# 3. install Container Runtime by downloading binary packages
you need to download containerd, runc, CNI (Container Netwrok Interface) binaries and install them.
#### containerd
> download from https://github.com/containerd/containerd/releases

Download the `containerd-<VERSION>-<OS>-<ARCH>.tar.gz` archive and extract it under `/usr/local`:
```
tar Cxzvf /usr/local containerd-1.6.2-linux-amd64.tar.gz
```

to start containerd as a service also download the `containerd.service` unit file:
```
cd /usr/local/lib/systemd/system
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
```
and the run:
```
systemctl daemon-reload
systemctl enable --now containerd
```

#### runc
> download https://github.com/opencontainers/runc/releases

Download the `runc.<ARCH>` and run:
```
install -m 755 runc.amd64 /usr/local/sbin/runc
```

#### CNI
> download https://github.com/containernetworking/plugins/releases

Download the `cni-plugins-<OS>-<ARCH>-<VERSION>.tgz` archive and run:
```
mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.1.1.tgz
```

> [!NOTE]
> ctr which is the CLI tool to intract with containerd is bundled with the containerd itself and you can `ctr version` to verify it.

### setting up systemd cgroup for containerd 
generate the default config file:
```
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

set the systemd cgroup driver in `/etc/containerd/config.toml` by editing the parameter `SystemdCgroup` value and changing it from `false` to `true`:
```
vim /etc/containerd/config.toml
    > search for SystemdCgroup
    > change to true
```

after the change restart:
```
sudo systemctl restart containerd
```

> [!NOTE]
> to configure the cgroup for kubelet, kubeadm allows you to pass `KubeletConfiguration` structure durring `kubeadm init` but in higher versions the default cgroup driver in kubeadm (v.1.22 and later) is systemd by default. <br>
> after you initialized the cluster you can check the cgroup driver for kubelet in `/var/lib/kubelet/config.yaml` dir, looking for the parameter `cgroupDriver: systemd`.



# 4. install kubectl
download the latest binary release with:
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

the command below will copy the downloaded file to `/usr/local/bin` with parameters:
 - `-o` = owner root
 - `-g` = group owner root
 - `-m` = set file's permission to 755
```
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

if you do not have root access install it in `~/.local/bin`:
```
$ mkdir -p ~/.local/bin
$ mv ./kubectl ~/.local/bin/kubectl
$ chmod +x ~/.local/bin/kubectl
```
and add the location to path
```
$ export PATH="$HOME/.local/bin:$PATH"
```

verify installation:
```
kubectl version --client
```



# 5. install kubeadm and kubelet
#### least requirements for all nodes:
- 2G RAM
- 2 CPU cores
- network connectivity between all machines
- unique hostname, MAC addrss and product_uuid for every node

> [!WARNING]
> the target distro has to provide `glibc` <br>
> and also make sure of your specific OS version, and kernel compatibility.

#### check MAC_ADDRESS and product_uuid to be unique for each node
k8s use MAC_address and product_uuid to uniquely identify the nodes. check the MAC address of each node by one of these commands. they have to be unique:<br>
`ip link` or `ifconfig -a`<br>

check the product_uuid of each node:
```
cat /sys/class/dmi/id/product_uuid
```

#### chcek required port
check the k8s port (6443) to be open by firewall and test it.<br>
on master node:
```
nc 127.0.0.1 6443 -zv -w 2
```
on worker nodes:
```
nc [MASTER_NODE_IP] 6443 -zv -w 2
```

#### check swap
kubelet will fail if the swap is on. you can tolerate the swap by adding `failSwapOn: false` to kubelet configuration. or just turn it off:
```
sudo swapoff -a
```

check swap. all output values or the row swap have to be zero:
```
free -m
``` 

#### installing
installing kubeadm and kubelet from official kubernetes repository --> `pkgs.k8s.io` (debian based distros) also in my case this is specific to Kubernetes 1.36:
```
$ sudo apt update
$ sudo apt install -y apt-transport-https ca-certificates curl gpg

$ sudo mkdir -p /etc/apt/keyrings
$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.36/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
$ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

$ sudo apt update

# you can also install kubectl here too
$ sudo apt install -y kubelet kubeadm

# hold these packages from being updated automatically 
$ sudo apt-mark hold kubelet kubeadm kubectl

$sudo systemctl enable --now kubelet
```



# 6. initialize the control plane
first set the hostnames for all nodes and also add the ip and hostname of all nodes including master node it self to master node's `/etc/hosts`<br>

in the master node initialize the cluster by this command:
```
kubeadm init \
  --apiserver-advertise-address=192.168.55.110 \
  --pod-network-cidr=10.244.0.0/16
```
- `--apiserver-advertise-address` = specifies the advertise address which is the master node's ip address
- `--pod-network-cidr` = this specify's the pods network ip which depends on what Pod Network Add-on you use(Flannel in this case) 
<br>
the command above will pull the control plane images and initialise the cluster and also guides you to run this sequence:
```
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```



# 7. deploy a pod network add-on
this will be flannel. deploy flannel:
```
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```


to confirm the isntallation of the network add-on to cluster, you can check the CoreDNS pod is running in the output of this command:
```
kubectl get pods --all-namespaces
``` 



# 8. join node-1 and node-2 to the cluster
to add worker nodes simply copy copy the `kubeadm join` command at the end of the output put of your previouse `kubeadm init` command and run it the target worker node you want to join the cluster.<br>
this is the syntax:
```
sudo kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>
```
example:
```
kubeadm join 192.168.55.110:6443 --token csdgbr.s5j1jvbhr3hs20mx \
        --discovery-token-ca-cert-hash sha256:072d70b1eee7fc86f7180879eca12d89b304680aff7f3d1784e52c78ef8a991c 
```

if you dont have the token or lost it just run this in master node:
```
sudo kubeadm token list
```

if the token is expired create new:
```
sudo kubeadm token create
```

or more handy:
```
sudo kubeadm token create --print-join-command
```

to get value of `--discovery-token-ca-cert-hash`:
```
sudo cat /etc/kubernetes/pki/ca.crt | openssl x509 -pubkey  | openssl rsa -pubin -outform der 2>/dev/null | \
   openssl dgst -sha256 -hex | sed 's/^.* //'
```

to confirm the node joined the cluster run this in control plane:
```
kubectl get nodes
```



# 9. how to configure kubectl for yourself to access cluster
by default worker nodes only require kubelet and container runtime to operate and we dont configure kubectl on them cause there is the risk cluster gettign compromised and in some cases not even the control plane needs kubectl cause of the risk.<br>
good practice is to install kubectl in a secured vm that can see cluster.

#### install kubectl binary
> easy copy commands from https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

verify kubectl installation
```
kubectl version --client
```

#### kubeconfig
in the control plane there is configuration file `/etc/kubernetes/admin.conf` which is a critical configuration file (kubeconfig) that grants full administrative access to k8s cluster. It is automatically generated on a control-plane node when you bootstrap a cluster using the kubeadm init tool.<br>

#### grand access
copy `/etc/kubernetes/admin.conf` in master node to the user's home dir of the machine that you want to control the cluster with.<br>
secured machine:
```
mkdir ~/.kube
```
master node:
```
scp /etc/kubernetes/admin.conf USER@SECURED_MACHINE_IP:~/.kube/config
```
secured machine (check access):
```
kubectl get all
```






## sources
> - source https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd
> - source https://github.com/containerd/containerd/blob/main/docs/getting-started.md
> - source https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
> - source https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
> - source https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/adding-linux-nodes/
> - source https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/