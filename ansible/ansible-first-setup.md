# on control node
1. create a user with sudo privillage

2. generate ssh keypair on control node, copy public-key to all managed nodes in `authorized_keys` file to stablish the connection. or use `ssh-copy-id` command. 

3. add the ansible repositoy:<br>
`sudo apt-add-repository ppa:ansible/ansible`<br>
`sudo apt update`

4. install ansible:<br>
`sudo apt install ansible -y`

5. open the default ansible inventory:<br>
`sudo nano /etc/ansible/hosts`<br>
add this as an example (replace VM_IP with your managed hosts IP):
```
[servers]
server1 ansible_host=VM_IP
server2 ansible_host=VM_IP

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```
check the default inventory file:<br>
`ansible-inventory --list -y`

6. ping the dest hosts by ansible (replace the user_name):<br>
`ansible all -m ping -u user_name`