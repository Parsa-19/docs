# ASLR, KASLR and KARL.<br>
they are all security techniques to mitigate attacks related to kernel.

## ASLR
ASLR or Address Space Randomization Layout is the security feature built into the OS that randomizes memory location of main system applications/processes, shared librarires, stack and heap memoy space of those applications.<br>
It prevents attacker to access function's return code of the loaded apps, to be able to load their payload right into the system or even it could be through buffer overflow in which attacker tries to overwrite applications data through overwriting the buffer (maybe aimed to change the function's return code to address to desired malicious code).even if the attacker tries the brute force to find that specific memory address it will cause the process to crash each time it addresses the worng place, and also could cause alert of suspicious activity if SOC teams consider such these things.<br>
Each time the OS reboots, memory location addressses of processes, stack and heap memory space and shared libraries are relocated at a different randomized location. 
> [!NOTICE]
> Memory addresses are chosen once when a new process is created (specifically during execve()), and they remain fixed for the lifetime of that process.

### enable and configuration of ASLR
ASLR is controlled via `randomize_va_space` kernel parameter.
```
sysctl kernel.randomize_va_space
```
this parameter has three levels that could be set as its value:
- `0` = ASLR is disabled and all memory regions are fixed addresses.
- `1` = ASLR is enabled on stack memory space, shared libraries, VDSO but not on heap.
- `2` = ALSR is fully enabled on stack and heap memory spaces, shared libraries and VDSO.
configure:
```
# make persistant accross the reboot
$ echo 'kernel.randomize_va_space = 2' | sudo tee /etc/sysctl.d/99-aslr.conf

# Apply via sysctl.conf or /etc/sysctl.d/*.conf system
sudo sysctl --system
```

## KASLR
KASLR or Kernel Address Space Layout Randomization is simply adding ASLR feature on kernel at boot time when it is about to load to memory.<br>
It means that the kernel code location in memory is randomized each time **(it only gets a new random memory address per each reboot)**.<br> 
because of one time address randomization per each time boot, its been a bit of debaits about how safe KASLR is.<br> 
because rebooting the system is not something you could do regularly to ensure the security so if an attacker could find memory leak, he could take advantage of this consistant and bypass the KASLR. although the nature of ASLR is at start time or in KASLR at boot time and this is how it works.

two methods for by-passing the KASLR (or in general the ASLR):
- brute force the address location which is noisy and identifiable by security teams due to multiple times of attempts and causing the restart or reboot of system.
Each time attcker try a new address it could be a location of random part of memory and unralated to what application address is targeted on leading to cause a crash to kernel and eventually the reboot.
- leak one pointer (memory leak) works like when the attacker knows the kernel binary for example the base 
