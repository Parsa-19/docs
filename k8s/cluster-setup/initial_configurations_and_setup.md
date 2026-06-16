# my general idea of the whole thing
what I went through when decided I need to know how to create cluster:


## network configuratoin 
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


## cgroups configuration
cgroups are used to constrain resources that are allocated to processes. Both kubelet and container runtime needs to access system cgroups to enforce resource management for pods and containers. They also have to use the same cgroup driver to work properly.<br>
there are two cgroup drivers:
- cgroupfs 
- systemd

#### cgroupfs
the default for kubelet but it is not recommended when systemd is the init system.

#### systemd
in systemd the init process generates and consumes the root cgroup and act as cgroup manager.

you need to make sure that both kubelet and container runtimes are using the same cgroup driver.


## install kubectl
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




## install Container Runtime by downloading binary packages
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

> [NOTICE]
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

> [NOTICE]
> When using kubeadm, manually configure the cgroup driver for kubelet.






# sources
> source https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd
> source https://github.com/containerd/containerd/blob/main/docs/getting-started.md