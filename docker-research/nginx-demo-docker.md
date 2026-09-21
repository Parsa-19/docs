- create a base dir:
	```
	mkdir nginx-demo
	```
- pull nginx by alpine tag:
	```
	docker pull nginx:alpine
	```
- create Dockerfile and put this in it:
	```
	FROM nginx:alpine
	COPY index.html /usr/share/nginx/html
	EXPOSE 80
	CMD ["nginx", "-g", "daemon off;"]
	```
- create index.html and put this in it:
	```
	<html>
		<h1>HELLO FROM NGINX ALPINE CONTAINER<h1>
	</html>
	```
- build the Dockerfile to image:
	```
	docker build -t nginx-costume-image ./
	```
- run the created image to container (in background):
	```
	docker run -d --name nginx-container1 -p 80:80 nginx-costume-image
	```
- see the container is running with no problem:
	```	
	docker ps
	```
