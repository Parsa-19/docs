# SAK (Security Attention Ket) also known as SAS (secure attention sequence):
is a security feature that protects you from fake or spoofed login applications that targets on stealing your password. It is a sequence of keys that is only recognizable by the kernel.

for example a famous SAK is `Ctrl + Alt + delete` 

this works as a key combination that act as key intrupts or signals straight to the kernel and kills all user processes attached to the current terminal. so the kernel know there was no malicious application in between and safely displays the login screen.

users should do it before the login to OS.

when you press the Secure Attention Key the kernel will immidietly get the key event and  never go through the user-space. thus no application in between can seet and act as login screen. kernel then find every process attached, send SIGKILL and destroys them. you can see the fresh login screen in terminal then.

> [!NOTICE]
> when you press Secure Attention Key it will go throught the processes related to you current tty and other processes continues running. 
