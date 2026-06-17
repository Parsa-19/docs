# my general idea of the whole thing
what I went through when decided I need to know how to create cluster:
### this example scenario:
create a k8s cluster between three ubuntu nodes using kubeadm. <br>
network configuration is: <br> 
two network adapters for each vm (I am implementing this in virtual box) one adapter is NAT (10.0.2.0/24) and one is hostonly (192.168.55.0/24).

## 1. network configuratoin 
set the ip_forward parameter:
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF
```
<br>
apply immidiatly without reboot:

`sudo sysctl --system`

<br>
verify:

`sysctl net.ipv4.ip_forward`


## 2. cgroups configuration
cgroups are used to constrain resources that are allocated to processes. Both kubelet and container runtime needs to access system cgroups to enforce resource management for pods and containers. They also have to use the same cgroup driver to work properly.<br>
there are two cgroup drivers:
- cgroupfs 
- systemd

#### cgroupfs
the default for kubelet but it is not recommended when systemd is the init system.

#### systemd
in systemd the init process generates and consumes the root cgroup and act as cgroup manager.

you need to make sure that both kubelet and container runtimes are using the same cgroup driver.


## 3. install Container Runtime by downloading binary packages
you need to download containerd, runc, CNI(Container Netwrok Interface) binaries and install them.
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

> [!NOTICE]
> ctr which is the CLI tool to intract with containerd is bundled with the contained itself and you can `ctr version` to verify it.

### setting up containerd for kubernetes
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

> [!NOTICE]
> When using kubeadm, manually configure the cgroup driver for kubelet.


## 4. install kubectl
download the latest binary release with:
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

copy the downloaded file with owner root (-o) and group root (-g) and permission set of the file with 755(-m) to `/usr/local/bin` by this command:
```
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

if do not have root access install it in `~/.local/bin`:
```
mkdir -p ~/.local/bin
mv ./kubectl ~/.local/bin/kubectl
chmod +x ~/.local/bin/kubectl
# then add the location to path
export PATH="$HOME/.local/bin:$PATH"
```

verify installation:
```
kubectl version --client
```


## 5. install kubeadm and kubelet
least vm requirements:
- 2G RAM
- 2 CPU cores
- network connectivity between all machines
- unique hostname, MAC addrss and product_uuid for every node
> [!WARNING]
> the target distro hase to provide `glibc`

make sure of your specific os version and kernel compatibility.

k8s use MAC_address and product_uuid to uniqely identify the nodes. check the MAC address of each node. they have to be unique:
```
ip link
/
ifconfig -a
```
check the product_uuid of each node:
```
cat /sys/class/dmi/id/product_uuid
```

kubelet will fail if the swap is on. you can tolerate the swap by adding `failSwapOn: false` to kubelet configuration. or just turn it off:
```
sudo swapoff -a
```

installing kubeadm and kubelet from official kubernetes repository --> `pkgs.k8s.io` (debian based distros) also this is specific to Kubernetes 1.36:
```
$ sudo apt update
$ sudo apt install -y apt-transport-https ca-certificates curl gpg

$ sudo mkdir -p /etc/apt/keyrings
$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.36/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
$ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

$ sudo apt-get update

# you can also install kubectl here too
$ sudo apt-get install -y kubelet kubeadm

# hold these packages from being updated automatically 
$ sudo apt-mark hold kubelet kubeadm

$sudo systemctl enable --now kubelet
```

# 7. initialize the control plane
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

# 8. deploy a pod network add-on
this will be flannel. deploy flannel:
```
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```


to confirm the isntallation of the network add-on to cluster, you can check the CoreDNS pod is running in the output of this command:
```
kubectl get pods --all-namespaces
``` 

# join node-1 and node-2 to the cluster
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


## sources
> - source https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd
> - source https://github.com/containerd/containerd/blob/main/docs/getting-started.md
> - source https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
> - source https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
> - https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/adding-linux-nodes/
> - https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/