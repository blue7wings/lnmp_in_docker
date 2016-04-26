#Linux + Nginx + PHP + Mysql environment in Docker
#How to use
```
   docker-compose up -d --build
```
use `docker-machine` command to check out your ip, and enter `ip:80` to visit `index.php` and `ip:8080` to visit phpMyAdmin
#Caution
if you not in China, please REMOVE `daocloud.io` in `docker-compose.yml` and all of `Dockerfile`
