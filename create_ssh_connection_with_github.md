## create a connection between your repo in ubuntu server and your github account

you need to create ssh key pair. add the private ssh key to ssh agent and add public key to your github account. <br><br><hr>  

install openSSH first <br>
`sudo apt install openssh-server`

generate ssh key pairs<br>
`ssh-keygen -t ed25519 -C "your_email@example.com"`

Start the ssh-agent in the background <br>
`eval "$(ssh-agent -s)"`

Add your SSH private key to the ssh-agent <br>
`ssh-add ~/.ssh/id_ed25519`

Copy your public key <br>
`cat ~/.ssh/id_ed25519.pub`

the go to your github account. <br> 
click your profile > settings > SSH & GPG keys > new ssh key <br>
paste the public key here and save it <br>

finally authenticate the connection <br>
`ssh -T git@github.com`