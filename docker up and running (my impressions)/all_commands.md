```
docker image build -t example/docker-node-hello:latest .
docker build -t example/docker-node-hello:latest .
docker build --build-arg email=me@example.com -t example/docker-node-hello:latest .

docker container run --rm -d -p 8080:8080 example/docker-node-hello:latest
docker run --rm -d -p 8080:8080 example/docker-node-hello:latest

docker container ls
docker ps

docker context list
docker image inspect example/docker-node-hello:latest | grep maintainer

docker stop CONTINAER-ID
docker rm CONTAINER-ID
docker image ls

docker login
# login info will be saved in ${HOME}/.docker/config.json

docker logout

docker login someregistry.example.com

docker image tag example/docker-node-hello:latest docker.io/${<mydockerhubuser>}/docker-node-hello:latest

docker image push ${<mydockerhubuser>}/docker-node-hello:latest

docker image pull ${<mydockerhubuser>}/docker-node-hello:latest

docker logs <containerName>

docker container export ddc3f61f311b -o web-app.tar

docker container run --rm -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh

docker image tag IMAGE-NAME NEW-TAG(NAME)

docker image history IMAGE-TAG

time docker image build --no-cache .
```

### looking into the container’s filesystem while the container doesnt contain a shell or SSH.
in this situation you cant even do docker exec. <br>
to connect directly to the Docker server and then look into the container’s filesystem there is a directory:<br>
`/var/lib/docker/rootfs/overlayfs/`<br>
which contains direcotries in it that named after their IDs. when you open that you'll see container's filesystem.

an example of such containers clone this repo and build and run the image: <br>
```
git clone https://github.com/spkane/scratch-helloworld.git
docker build -t go-app .

# run and test container
docker run --rm -d -p 8080:8080 go-app
curl 127.0.0.1:8080
docker inspect <CONTAINER-ID>

# to get to the file system you need to be root
sudo su -
cd /var/lib/docker/rootfs/overlayfs/CONTAINER_ID/
ls

# you'll see the filesystem
```
if you dont have root access you can use a debian container and get to the filesystem from there: <br>
`docker container run --rm -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh`<br>
then:<br>
`cd /var/lib/docker/rootfs/overlayfs/CONTAINER_ID/`
`ls`
