# SSH to local VM

you have to install ssh server on vm os: <br>
`[vm]# sudo apt install openssh-server`

start it out: <br>
`[mv]# sudo systemctl start ssh`

check its starting status: <br>
`[vm]# sudo systemctl status ssh`

and make sure that you have ssh client in host machine: <br>
`[host]# sudo apt install openssh-client`

then find out your vm's ip address by: <br>
`[vm]# ip addre`

ssh to vm from host: <br>
`[host]# ssh <vm-user-name>@<vm-ip-address>`


you'll be prompt to type your password. its tha one you set for vm user.
you'r in then..
