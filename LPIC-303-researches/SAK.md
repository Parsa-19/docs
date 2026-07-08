# SAK (Security Attention Key) also known as SAS (secure attention sequence):
is a security feature that protects you from fake or spoofed login applications that targets on stealing your password. It is a sequence of keys that is only recognizable by the kernel.

for example a famous SAK is `Ctrl + Alt + delete` (a standard SAK on windows systems)

this works as a key combination that act as key intrupts or signals straight to the kernel and kills all user processes attached to the current terminal session. so the kernel know there was no malicious application in between and safely displays the main login screen.

users should do it before the login to OS.

when you press the Secure Attention Key the kernel will immidietly get the key event in kernel space and has nothing to do with the user-space. thus no application in between can seet and act as login screen. kernel then find every process attached to the current session and send SIGKILL and destroys those extra processes. you can see the fresh login screen in terminal then.

> [!NOTICE]
> when you press Secure Attention Key it will go throught the processes related to your current tty and other processes continues running.

In linux systems when you press SAK sequence, kernel instantly executes the virtual terminal's SAK handler (`do_SAK()`) which will:
 1. identify your current terminal (TTY). 
 2. kills all associated processes to that terminal session.
 3. init or systemd spins up the main login prompt.

The implementation resides entirely within the Linux kernel's virtual terminal (VT) subsystem and is not performed via a normal system call. Instead, it is initiated through the keyboard input path (either the keyboard driver or the SysRq handler), which directly invokes the kernel's SAK implementation.

## tiggering SAK in Linux

as described above in Linux SAK is implemented as a kernel-level console action. Unlike Windows SAK, Linux does **not** reserve a universal hardware key combination. instead, SAK is available only on virtual terminals (TTYs) and must be explicitly triggered through one of the following mechanisms.

### 1. Magic SysRq

The simplest method is the Magic SysRq interface:

first enable the prameter using:
```
sysctl -w kernel.sysrq=1
```

then press:
```
Alt + SysRq + K
```

you will see the fresh login page.

The `K` in SysRq command invokes the SAK handler for the current virtual console.

Internally, the keyboard driver recognizes the SysRq sequence and dispatches the command directly to the kernel's SysRq handler, which executes the SAK routine for the active TTY.

### 2. Keyboard layout mapping (traditional method)

SAK can also be bound to an arbitrary key combination using the kernel keymap.

A common example is mapping it to `Ctrl + Alt + Pause`:

```sh
echo "control alt keycode 101 = SAK" | /bin/loadkeys
```

or equivalently in a keymap file:

```text
control alt keycode 101 = SAK
```

After loading the keymap with `loadkeys`, pressing the configured key combination causes the keyboard driver to generate the special `SAK` action instead of a normal key event.

> [!NOTE]
> The keycode is hardware-dependent. `101` commonly corresponds to the Pause key, but the correct value should be determined with tools such as `showkey`.

> [!IMPORTANT]
> The SAK action only works on Linux virtual consoles (TTYs). It does **not** operate within graphical desktop sessions (X11 or Wayland), terminal emulators, or SSH sessions.

> [!NOTICE]
> Recent versions of systemd (v257 and later) introduced `Ctrl + Alt + Shift + Esc` as a convenient shortcut that invokes the Linux Secure Attention Key on supported systems. This is implemented in user space by systemd and ultimately requests the kernel to perform the SAK operation on the active virtual console.

