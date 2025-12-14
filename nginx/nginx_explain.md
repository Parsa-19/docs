## general nginx
this is a high performance web server which handles http requests. <br>

nginx is used as:
- mainly web server which response really fast to http requests
* reserve proxy 
+ used with slower upstream servers such as Unicorn or uWGI
- load-blancing : distribute the traffic in disired way

the basic artichcture of nginx is consists of two components: <br>
**master process** & **worker processes** <br>
-master process: read the configurations and maintain worker processes. <br>
-worker process: do the actual processing of requests.

the way nginx and its modules work are determind on `/etc/nginx/nginx.conf` file.

## add nginx package-repo and installing nginx on ubuntu

## start stop reload and basics controls on nginx
you can control nginx using systemctl, sysVinit, or signals:
### using systemctl
```
systemctl start nginx
systemctls stop nginx
systemctl restart nginx
systemctl reload nginx
systemctl enable nginx
systemctl disable nginx
```
reload just reloads the configuration while restart would restart the whole thing. <br>
enable and disable are used to manage nginx to be run automatically durring boot time or not.

### use nginx command & signals
```
nginx
nginx -s stop
nginx -s quit
nginx -s reload 
nginx -t
```
stop will shut down nginx imidietly but quit wait for worker process to finish the rquests and then shut down it. <br>
-t will check if the configurations are ok or not. <br>
or do it like:
```
sudo /etc/init.d/nginx start
sudo /etc/init.d/nginx restart
sudo /etc/init.d/nginx stop
sudo /etc/init.d/nginx reload
```

<br>
you can do the basics with all these commands.

## nginx configuration structure

## ginx logs

