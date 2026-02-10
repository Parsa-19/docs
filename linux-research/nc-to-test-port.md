# objective
you can use nc as alternative to telnet to test the port or connection between two VMs:

## client/server test

listen on a specific port on VM (1):<br>
`VM-1# nc -lv 1234`
- l : listen
- v : verbose

make a connection from VM (2):<br>
`VM-2# nc -v [VM_1_IP] 1234`

test range of ports to VM (1):<br>
`VM-2# nc -zv 2.144.22.198 1230-1235`

## as regular ping comand (with port test)
`nc -zv google.com 443`

