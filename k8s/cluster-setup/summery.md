
all nodes:

    swap off 
    ipforward on
    check MAC address or /sys/class/dmi/id/product_uuid on each node to make sure each are unique

    set CRI's cgroup to systemd - kubelet cgroup will be systemd by default if not set
    CRI installation (includes containerd runc and cni-binaries) 
