### ptrace 
this is a system call that enables a process (tracer) to trace, observe and control another process (tracee) execution which is used for program debugging purposes and also in T-shooting tools.

due to the power of this system call it could be in the hands of bad people. for example an unprivileged user could use ptrace to find the content of application's memory and memory leak at the end.  

### kernel.yama.ptrace_scope
it determines that what process can be traced and debugged by ptrace.

default value:
```
sysctl kernel.yama.ptrace_scope
```

values that could be set on this parameter:
- `0` -> processes can be debugged as long as the tracer and tracee has the same uid (DEFAULT)
- `1` -> only parent process can be debugged   
- `2` -> only admin is allow to use ptrace (CAP_SYS_PTRACE capability is required)
- `3` -> no process is allowd to use prace and to re-enable ptrace again, a reboot is needed.

to set:
```
sysctl kernel.yama.ptrace_scope=1 >> /etc/sysctl.d/ptrace_scope_set.conf
```