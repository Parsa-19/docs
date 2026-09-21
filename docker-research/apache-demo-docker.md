# a docker file for apache

```
FROM ubuntu:latest

MAINTAINER p4r54-apache

RUN apt-get update && apt-get install -y apache2 && apt-get clean && rm -rf /var/lib/apt/lists/*

ENV APACHE_RUN_USER=www-data
ENV APACHE_RUN_GROUP=www-data
ENV APACHE_LOG_DIR=/var/log/apache2

EXPOSE 80

CMD ["/usr/sbin/apache2", "-D", "FOREGROUND"]
```

> [!NOTE]
> save the file by name of "Dockerfile"
> RUN arguments, runs inside the pulled ubuntu container
> EXPOSE indicates that the container is listening on port 80 (and doesnt relate to host machine you will specify that later)

the procedure will be like:
Docker-file <--(build)--> docker image <--(run)--> container

steps:

- enter the directory of your docker file.
- build docker-file:
	```
	docker build -t apache-image-name ./
	```
- specify the name of your image and location of the docker-file
	```
	docker images
	```
	> you can see apache-image-name image is created
- run the image to create a container:
	```
	docker run -p 8000:80 --name=apache_container1 apache-image-name
	```
	> check the browser to see apache
- in another terminal see the container running:
	```
	docker ps
	```
- stop the container:
	```
	docker stop container_ID
	```
