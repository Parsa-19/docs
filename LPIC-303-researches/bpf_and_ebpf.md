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


## how does ebpf works and the concepts

### hook 







> [!NOTICE]
> for more information on how it works refer to main documentation: `https://ebpf.io/what-is-ebpf/#maps`

