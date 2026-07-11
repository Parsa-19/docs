# bpf and ebpf overview
ebpf is a mechanism that can run small custom programs to add desired additional capabilities to kernel at runtime.
 
it ensures the safety an efficiency of the program and the code so you dont need to wirte kernel modules for customization and/or hardcode your program in kernel source code (more than 30m lines of code) and recompiling it. 

programs that uses ebpf natively are compiled with the aid of a Just-In-Time (JIT) compiler and verification engine.

you can easily use bpf/ebpf functionalities inside the kernel in kernel ralated programs. 

## bpf
traditionally bpf was just for packet filtering in level of kernel. bpf is used as an enhancment to make the procedure faster. so it wont be like kernel takes the entry packet, pass it to user-space application and then sends it back again to kernel for calculations. we dont want that. instead bpf lets you to upload a tiny filtering program that seets in kernel and decides about packets to be accepted or rejected at the time. tools like `tcpdump` use bpf.

## ebpf
now ebpf (extended bpf) can do a lot more in kernel and is considered as general-purpose kernel programming feature which the kernel programs can use it to:
- observe system calls
- inspect network traffic
- trace function calls
- monitor file access
- collect performance metrics
- enforce security policies
all without writing kernel modules.

> [!NOTICE]
> projects like **Cilium** uses ebpf that k8s uses this as its network add-on to manage pod network.

### overal use cases
in performance, observibility, security, Storage.. thats huge!


## ebpf simple examples
### example 1: ebpf program that is tracing systemcalls attached to `open()` to open a simple file by any process

for example I need to know when a process opens a file:
- without epbf:
    - application --> system call --> kernel open()
- with ebpf:
    - application --> system call --> open() --> ebpf program executes --> collect filename --> store into map --> kernel continues normally

consider firefox wants to open a file. ebpf here is tracing system calls that are attached to `open()` requested by firefox:

firefox --> open("/etc/passwd") --> ebpf program in kernel executes --> send an event to user-space

the event could contain information for example:
```
PID = 1234
Process = firefox
File = /etc/passwd
Timestamp = 10:34:21
```

### example 2: packet filtering 
another example can be considered as packet filtering scenario;

suppose these packets are arrived at kernel's entry point:
```
Packet A: TCP
Packet B: UDP
Packet C: TCP
Packet D: ICMP
```

an ebpf program in kernel after its related hook is triggerd be like:
```
if (packet.protocol == TCP)
    return PASS;

return DROP;
```
kernel will execute this for each packet.

as a result it will only accpets tpc packets and reject rest of them:
```
Packet A: TCP
Packet C: TCP
```

no user-space involvment is needed here.


# epft ensures safty (Verifier)
epbf uses a verifier to ensure the safty of program in kernel. running and arbitrary code inside the kernel would be dangerous. the code could be destructive by purpose or unintentionally.<br>
for example if you upload:
```
while(1){}
```
kernek will freeze.

the verifier will check:
- every loop terminates
- no invalid pointer access
- no kernel memory corruption
- no use of uninitialized values
- stack limits are respected
- all execution paths are safe

and reject the code if it is unsafe.

## the procedure (what happens)
```
Write eBPF Program
        │
        ▼
Compile
        │
        ▼
Load into Kernel
        │
        ▼
Verifier checks it
        │
        ▼
JIT Compiler (optional)
        │
        ▼
Attach to Hook
        │
        ▼
Kernel Event Happens
        │
        ▼
eBPF Program Runs
        │
        ▼
Read/Write Maps
        │
        ▼
Send Event to Userspace
        │
        ▼
Userspace Reads Event
        │
        ▼
Display / Analyze / Store
```

## mirigate unprivileged access to ebpf
do not allow unprivileged users to use ebpf in linux kernel by enabling this kernel parameter:
```
echo "kernel.unprivileged_bpf_disabled=1" >> /etc/sysctl.d/bpf_unprivileged_access.conf
```
apply:
```
sysctl -p /etc/sysctl.d/bpf_unprivileged_access.conf
```



> [!NOTE]
> for more information on how it works refer to main documentation: `https://ebpf.io/what-is-ebpf/#maps`

