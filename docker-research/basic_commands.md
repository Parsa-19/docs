`docker --version`
`docker --help`

`docker run -it ubuntu:latest bash`
`docker run -d -p 8080:80 docker/welcome-to-docker`
`docker run -d --name dock_container dock-img`
`docker ps -a`

`docker search docker/welcome-to-docker`
`docker pull docker/welcome-to-docker`
`docker image ls`
`docker images`
`docker image history ubuntu:latest`

`docker start -ia container`
`docker stop container`

`docker rm container`
`docker rmi image`
`docker build -t new-container .`
`docker build -t new-container -f mydockerfile .`
`docker system prune`

`docker logs container`

`docker login`
`docker push`
`docker kill`
`docker exec`
`docker commit`
`docker import`
`docker export`
`docker container`
`docker compose`
`docker swarm`
`docker service`
