# install docker on Ubuntu
ensure no older version is installed:<br>
`sudo apt-get purge -y docker docker.io containerd runc docker-engine docker-ce docker-ce-cli docker-compose-plugin`<br>
`sudo apt-get autoremove -y docker docker.io containerd runc docker-engine docker-ce docker-ce-cli docker-compose-plugin`

install dependecies, create keyrings dir and add key and set its permissions, then add the repository in apt source.list.d:
```
# Add Docker's official GPG key:
sudo apt update
sudo apt install \
    ca-certificates \
    curl \
    lsb-release
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

install docker then:<br>
`sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`

> [!NOTE]
> to work with docker you need both **docker-client** and **docker-server**. what you installed is Docker Community Edition (docker-ce) which beside the docker client it also set up docker server too. so you dont need to set it up sepratly. you just need to ensure the docker server(dockerd) is running.

make sure docker service starts on every boot:<br>
`sudo systemctl enable docker`

start the service:<br>
`sudo systemctl start docker`