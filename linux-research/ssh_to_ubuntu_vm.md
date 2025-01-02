you have to install ssh server on vm os:
[vm]# sudo apt install openssh-server
start it out:
[mv]# sudo systemctl start ssh
check its starting:
[vm]# sudo systemctl status ssh

and make sure that you have ssh client in host machine:
[host]# sudo apt install openssh-client

then find out your vm's ip address by:
[vm]# ip addre
ssh to vm from host:
[host]# ssh <vm-user-name>@<vm-ip-address>

you'll be prompt to type your password. its tha one you set for vm user.
you'r in then..
