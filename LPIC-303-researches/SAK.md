# SAK (Security Attention Key) also known as SAS (secure attention sequence):
is a security feature that protects you from fake or spoofed login applications that targets on stealing your password. It is a sequence of keys that is only recognizable by the kernel.

for example a famous SAK is `Ctrl + Alt + delete` (a standard SAK on windows systems)

this works as a key combination that act as key intrupts or signals straight to the kernel and kills all user processes attached to the current terminal. so the kernel know there was no malicious application in between and safely displays the main login screen.

users should do it before the login to OS.

when you press the Secure Attention Key the kernel will immidietly get the key event and  never go through the user-space. thus no application in between can seet and act as login screen. kernel then find every process attached, send SIGKILL and destroys them. you can see the fresh login screen in terminal then.

> [!NOTICE]
> when you press Secure Attention Key it will go throught the processes related to your current tty and other processes continues running.

In linux systems when you press SAK sequence, kernel instantly executes `do_SAK()` which will:
 1. identify your current terminal (TTY). 
 2. kills all associated processes to that terminal session.
 3. init or systemd spins up the main login prompt.

## triggering SAK in linux
through two methods
1. sysrq: alt + sysrq + k
2. custom layout mapping (traditional way)

### set layout mapping
you do it through loadkeys and it is common to set it as `ctrl + alt + pause`.
```
echo "control alt keycode 101 = SAK" | /bin/loadkeys
```
> [!NOTICE]
> recent systemd versions such as systemd v257+ introduced this sequence to act as a reliable shortcut `Ctrl + Alt + Shift + Esc`.

