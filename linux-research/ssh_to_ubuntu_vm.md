# SSH to local VM

you have to install ssh server on vm os: <br>
`[remote-vm]# sudo apt install openssh-server`

start it out: <br>
`[remote-vm]# sudo systemctl start ssh`

check its starting status: <br>
`[remote-vm]# sudo systemctl status ssh`

and make sure that you have ssh client in host machine: <br>
`[host]# sudo apt install openssh-client`

then find out your vm's ip address by: <br>
`[remote-vm]# ip addre`

ssh to vm from host: <br>
`[host]# ssh <vm-user-name>@<vm-ip-address>`


you'll be prompt to type your password. its the one you set for vm user.
you'r in then..

## Implement SSH keys

generating ssh key pairs in client <br>
`[client]# ssh-keygen`

copy pulbic key "~/.ssh/id_rsa.pub" to server using one of these methods: <br>
1. by 'ssh-copy-id' (you need password based auth):
`[client]# ssh-copy-id username@remote_host`

2. copy it using ssh itself (you need password based auth):
`[client]# cat ~/.ssh/id_rsa.pub | ssh username@remote_host "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"`

3. copy manually, ensure the file and dir exists, has permissions and .ssh dir has user:user ownership:
`[remote-vm]# mkdir -p ~/.ssh`
`[remote-vm]# echo public_key_string >> ~/.ssh/authorized_keys`
`[remote-vm]# chmod -R go= ~/.ssh`
`[remote-vm]# chown -R parsa:parsa ~/.ssh`

test the connection: <br>
`[client]# ssh username@remote_host`

then just disable password auth on server: <br>
`[remote-vm]# sudo nano /etc/ssh/sshd_config`

add this line to config file: br
```
...
PasswordAuthentication no
...
```

restart ssh then to read config file:
`[remote-vm]# sudo systemctl restart sshd`