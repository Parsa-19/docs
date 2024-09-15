## 1. install docker-ce 
before installing docker you need to uninstall these packages:

	- docker.io
	- docker-compose
	- docker-compose-v2
	- docker-doc
	- podman-docker
uninstall these by:
	
	for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
now do:

	sudo apt update
then install some prerequisite packages to let apt use packages over https:

	sudo apt install apt-transport-https ca-certificates curl software-properties-common
add docker repository to APT sources:

	sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
Make sure you are about to install from the Docker repo instead of the default Ubuntu repo; then you can see docker is not insatlled and is going to be installed from docker repo:

	apt-cache policy docker-ce
then install docker:

	sudo apt install docker-ce
check docker engine status: 

	sudo systemctl status docker
----------------------------------------------------------------------------------------------------
## 2. install docker compose 
now that you have the docker repository added; install docker compose easily like this:

	sudo apt-get update
 	sudo apt-get install docker-compose-plugin
----------------------------------------------------------------------------------------------------
## 3. install php in server 
first: 

	apt update -y
	apt upgrade -y
install this:

	apt install software-properties-common
then add the repository for php:

	add-apt-repository ppa:ondrej/php
install php8.3:

	apt install php8.3 php8.3-cli php8.3-fpm
install additional PHP extensions:

	apt install php8.3-{mysql,curl,xsl,gd,common,xml,zip,xsl,soap,bcmath,mbstring,gettext}
check the version:

	php -v
uninstall any other version like:

	sudo apt purge php8.2*
----------------------------------------------------------------------------------------------------
## 4. create laravel project and its dependencies 
go in home directory and clone the laravel project:

	cd ~
	git clone https://github.com/laravel/laravel.git laravel-app
now we will install the dependencies of laravel project with docker to avoid isntall composer globally. so use docker's composer image to mount the directories:

	docker run --rm -v $(pwd):/app composer install
this will pull the composer image first(if you didnt have the image locally) then run that image to create the container.  
-v and --rm flags with docker run creates an ephemeral container that will be bind-mounted to your current directory before being removed.  
This will copy the contents of your ~/laravel-app directory to the container and also ensure that the vendor folder Composer creates inside the container is copied to your current directory.

> [!NOTE]  
> if you had problem runnig this command then install composer globally and create a laravel project by composer manually.
----------------------------------------------------------------------------------------------------
## 5. setting up containers using docker compose 
now we will create a docker compose file(.yml) to create all three containers and their configurations and data persistings. after that contaniers are managed by docker compose.  
two containers are images that will be pulled (nginx, mysql),  
one container is dockerfile witch will be built and then run to be a container and is not being pulled (php)

go to the project dir:
	
 	cd ~/laravel-app
create a yml file:

	nano docker-compose.yml
add this to it:
```
services:

  #PHP Service:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: php-costume-image
    container_name: app
    restart: unless-stopped
    tty: true
    environment:
      SERVICE_NAME: app
      SERVICE_TAGS: dev
    working_dir: /var/www
    networks:
      - app-network
    volumes:
       - ./:/var/www
       - ./php/local.ini:/usr/local/etc/php/conf.d/local.ini

  #Nginx Service: 
  webserver:
    image: nginx:latest
    container_name: webserver
    restart: unless-stopped
    tty: true
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./:/var/www
      - ./nginx/conf.d/:/etc/nginx/conf.d/
      - ./certbot/www/:/var/www/certbot/
    networks:
      - app-network

  #MySQL Service:
  db:
    image: mysql:8.0
    container_name: db
    restart: unless-stopped
    tty: true
    ports:
      - "3306:3306"
    environment:
      MYSQL_DATABASE: laravel
      MYSQL_ROOT_PASSWORD: mysql542
      MYSQL_ALLOW_EMPTY_PASSWORD: true
      SERVICE_TAGS: dev
      SERVICE_NAME: mysql
    networks:
      - app-network
    volumes:
      - ./dbdata:/var/lib/mysql
      - ./mysql/my.cnf:/etc/mysql/my.cnf

#Docker Networks
networks:
  app-network:
    driver: bridge

# Volumes
volumes:
  dbdata:
    driver: local
```
	
this will create all three containers.  
in app block we had addressed the dockerfile witch we will create later.  
all the network and valumes that you see in this file are related to data persistings and will be fixed later
	
- PHP Service: This service definition contains the Laravel application and runs a custom Docker image which name is php-costume-image. hole project is copied in /var/www inside container(bind-mounting)  
- Nginx Service: This service definition pulls the nginx:latest image from Docker and exposes ports 80 and 443
- MySQL Service: This service definition pulls the mysql:8.0 image. there is also some env variables that you need to set like password. This service definition also maps port 3306 on the host to port 3306 on the container.

----------------------------------------------------------------------------------------------------

## 6. persisting data for containers and create a new yml file
	(note: you can skip step 6 if you know data persistings. the yml file in previous step is edited with data persistings)
	
	by data persisting in docker you can communicate between directories and files in container and host
	you will make use of volumes and bind mounts for persisting the database, and application and configuration files.
	
	create a folder named dbdata to store the database data during the restart of container in laravel-app dir:
		>>> mkdir ~/laravel-app/dbdata
	add this to yml file to persist data of database:
...
#MySQL Service
db:
  ...
    volumes:
      - dbdata: /var/lib/mysql
      - ./mysql/my.cnf:/etc/mysql/my.cnf
    networks:
      - app-network
...
	The named volume dbdata persists the contents of the /var/lib/mysql folder present inside the container
	
	also at bottom of yml file add:
...
#Volumes
volumes:
  dbdata:
    driver: local
    
	for nginx data persisting add:
#Nginx Service
webserver:
  ...
  volumes:
     - ./:/var/www
     - ./nginx/conf.d/:/etc/nginx/conf.d/
  networks:
      - app-network
      	
      	it is two valumes one for app code and another for nginx configuration
      	
      	after that add php data persisting:
#PHP Service
app:
  ...
  volumes:
      - ./:/var/www
      - ./php/local.ini:/usr/local/etc/php/conf.d/local.ini
  networks:
      - app-network    	
    
    also two valuems one for app code and another for php configuration
    
    
    note:you can see the edited yml file in previous step

----------------------------------------------------------------------------------------------------

## 7. create a php-docker-file 
	now we are going to create dockerfile for php. then thanks to the docker compose it will create the php-image and create php-container automatically when we run docker compose up with other containers
	
	go to laravel-app dir;
		>>> cd ~/laravel-app
	create the file:
		>>> nano dockerfile
	add this to it:
FROM php:8.3-fpm

# Set Environment Variables
ENV DEBIAN_FRONTEND=noninteractive

# Copy composer.lock and composer.json
COPY composer.lock composer.json /var/www/

# Set working directory
WORKDIR /var/www

#
#--------------------------------------------------------------------------
# Software's Installation
#--------------------------------------------------------------------------
#
# Installing tools and PHP extentions using "apt", "docker-php", "pecl",
#

# Install "curl", "libmemcached-dev", "libpq-dev", "libjpeg-dev",
#         "libpng-dev", "libfreetype6-dev", "libssl-dev", "libmcrypt-dev",
RUN set -eux; \
    apt-get update; \
    apt-get upgrade -y; \
    apt-get install -y --no-install-recommends \
            curl \
            libmemcached-dev \
            libz-dev \
            libpq-dev \
            libjpeg-dev \
            libpng-dev \
            libfreetype6-dev \
            libssl-dev \
            libwebp-dev \
            libxpm-dev \
            libmcrypt-dev \
            libonig-dev; \
    rm -rf /var/lib/apt/lists/*
    
RUN set -eux; \
    # Install the PHP pdo_mysql extention
    docker-php-ext-install pdo_mysql; \
    # Install the PHP pdo_pgsql extention
    docker-php-ext-install pdo_pgsql; \
    # Install the PHP gd library
    docker-php-ext-configure gd \
            --prefix=/usr \
            --with-jpeg \
            --with-webp \
            --with-xpm \
            --with-freetype; \
    docker-php-ext-install gd; \
    php -r 'var_dump(gd_info());'

# Install composer
#RUN curl -sS https://getcompo0ser.org/installer | php -- --install-dir=/usr/local/bin --filename=c>
#COPY --from=composer /usr/bin/composer /usr/bin/composer
#RUN composer self-update

# Add user for laravel application
RUN groupadd -g 1000 www
RUN useradd -u 1000 -ms /bin/bash -g www www

# Copy existing application directory contents
COPY . /var/www

# Copy existing application directory permissions
COPY --chown=www:www . /var/www

# Change current user to www
USER www

# Expose port 9000 and start php-fpm server
EXPOSE 9000
CMD ["php-fpm"]

	thats all we need for php container to have or run.

----------------------------------------------------------------------------------------------------

## 8. config php
	creating tha actuall directory for php config (according to data persisting in previous):
		>>> cd ~/laravel-app
		>>> mkdir php
		>>> nano php/local.ini
	add this to it:
upload_max_filesize=40M
post_max_size=40M
	
----------------------------------------------------------------------------------------------------

## 9. config nginx
	create dir:
		>>> mkdir -p ~/laravel-app/nginx/conf.d
	create file:
		>>> nano ~/laravel-app/nginx/conf.d/app.conf
	add this to it:
server {
    listen 80;
    listen [::]:80;

    index index.php index.html;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;

    root /var/www/public;

    server_name p4r54.ir www.p4r54.ir;
    server_tokens off;

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass app:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
    location / {
        try_files $uri $uri/ /index.php?$query_string;
        gzip_static on;
    }

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }
}

----------------------------------------------------------------------------------------------------

## 10. config mysql and create a user for it 
	create the dir:
		>>> mkdir ~/laravel-app/mysql
	create the file:
		>>> nano ~/laravel-app/mysql/my.cnf
	add this to it:
[mysqld]
general_log = 1
general_log_file = /var/lib/mysql/general.log
	
	then you need to create a user for mysql.
	to do that execute an interactive bash shell on the db container:
		>>> docker-compose exec db bash
	inside container log in to root account:
		(db)>>> mysql -u root -p
	enter the passwrod you had written in yml file and 
	
	see if you have laravel database that you had defined in yml file by:
		(db)>>> show databases;
	create the user:
		(db)>>> CREATE USER 'yourusername '@'localhost' IDENTIFIED BY 'yourpassword';
		(db)>>> GRANT CREATE, ALTER, DROP, INSERT, UPDATE, DELETE, SELECT, REFERENCES, RELOAD on *.* TO 'yourusername'@'localhost' WITH GRANT OPTION;
		(db)>>> FLUSH PRIVILEGES;

----------------------------------------------------------------------------------------------------

## 11. modifying .env file in laravel
	do :
		>>> cp .env.example .env
		>>> nano .env
	under "DB_CONNECTION" edit file like this:
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=laraveluser
DB_PASSWORD=your_laravel_db_password
	
	"DB_DATABASE" is the name if your database (which in previous we saw the database)
	assign "DB_USERNAME" by your username
	assign "DB_PASSWORD" by your db password

----------------------------------------------------------------------------------------------------

## 12. run the docker compose up finally and create containers 
	
	do docker compose up to create them all:
		>>> docker compose up -d --build
	check out containers:
		>>> docker ps
	just do these:
		>>> docker compose exec app php artisan key:generate
		>>> docker compose exec app php artisan config:cache


now if i was not wrong in any of those steps you should be able to see the laravel serve page by typing your ip/domain (whatever you had set in nginx config) in browser without ssl.









