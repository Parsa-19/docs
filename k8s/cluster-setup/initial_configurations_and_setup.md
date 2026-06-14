# my general idea of the whole thing
> source
> https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd

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
