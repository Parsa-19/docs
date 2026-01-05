# generall notes on docker images
> images are built out of dockerfile

> images are built up from individual layers

> you define a base image in docker file and then create new layers on that base image

> each line in docker file is a new layer on image 

> order matters in docker file. so on each build on the docker file, the lines that havent changed are already built so it will build the new image faster and as soon as it gets to the new line, the new layers will start to build. thats why you should add new changes to docker file at the end of file as possible so it takes less time to build each time.

> generally it is considered as best practice to try to run only a single process within a container

# dockerfile anatomy

a sample that demonstrate the general docker_file: 

FROM Ubuntu:latest

ARG email="this@this.com"

LABEL "maintainer"=$email

USER root

ENV working_dir /home/babe/project

RUN apt update -y

COPY ./config_files/* $working_dir

WORKDIR /data/app

CMD ["python3", "--version"]

# build and run image

### .dockerignore
you write the name of files and directories that you do not want the docker to include them durring the image build.<br>
the docker file doesnt sees them then.<br> 
e.g. :
```
.git
```
> book definition for .dockerignore file: in this file you define files and directories that you do not want to upload to the Docker host when you are building the image.

### build
to build image go to the root of project dir. <br>
this will build and tag an image based on the files in the current directory:<br>
`docker image build -t example/docker-node-hello:latest .`
> Using **docker image build** is functionally the same as using **docker build**

the dot "." at the end of the command points to the current directory and tells the docker what files to use to build image. thats why you put dockerfile and .dockerignore here.

> [!TIP]
> use **-t** to address docker file if it is not in the projects root dir.
> use **--no-cache** to disable cache durring the build.

if your system is running other processes you can also limit the resources available to your builds 

### run
run the built image:<br>
`docker container run --rm -d -p 8080:8080 example/docker-node-hello:latest`

this will run the container, remove it after stop, run in the background, forward local 8080 port to container 8080.

see the running containers:<br>
`docker container ls` or `docker ps`

shows docker context and it tells docker CLI (docker client) which docker daemon to use:<br>
`docker context list`

### build arguments
you can inspect the image and grep the maintainer: <br>
`docker image inspect example/docker-node-hello:latest | grep maintainer`

build with another arg value:<br>
`docker image build --build-arg email=me@example.com -t example/docker-node-hello:latest .`

### stop a container
`docker container stop CONTINAER-ID` or `docker stop CONTINAER-ID`

### a procedure to build image and test container
stop the existing container if there is one:<br>
`docker stop CONTAINER-ID`

remove that container:<br>
`docker rm CONTAINER-ID`

add your changes to dockerfile:<br>
`nano Dockerfile`

build a new layer on previous image: <br>
`docker build -t hello-node:latest .`

confirm:<br>
`docker image ls`

run the container in background and specify ports (removed when stoped):<br>
`docker run --rm -d -p 8080:8080 hello-node:latest`

confirm its up:<br>
`docker ps`

test the app:<br>
`curl http://127.0.0.1:8080`

### base image
for the base image you can:
- create your own (costumized to be very light weight)
- use the official distribution image (Ubuntu, Fedora, Debian)
- or use Alpine Linux (prepared and light weight)

### Storing Images
in development enviornment you create your docker file build it and create the image and run the contianer to to test it and repeat until you get your applocation up and ruuning.<br>
you need to store the image somwhere to access it in production environment or to save them in general.
