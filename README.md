原文地址：[http://www.blue7wings.com/php%20tutorial/Better-Dev-Envirenment-Docker.html](http://www.blue7wings.com/php%20tutorial/Better-Dev-Envirenment-Docker.html)

如果你看了Vagrant篇的内容，可能会想「我也不像做第一个装开发环境的人，把别人的镜像拿过来用就好了」，恭喜你，偷懒使人成长，你已经有成为开发大师的思路了，但是只不过有人稍早实现了你的idea，而且稍作改进成了现在的Docker，所以呐，有想法就去实现吧，不然就让别人占了先机咯（:

如果说Vagrant是将整个操作系统虚拟化，然后打包成镜像，分发使用，那么我们就可以把Docker简单理解为粒度更细的Vagrant，Docker可以将例如PHP，Nginx，MySQL这类服务打包成镜像，然后我们可以像拼积木一样组合他们，来实现我们想要的架构。

![架构图](http://ooyc2y4k2.bkt.clouddn.com/wiC0h)

## 安装Docker
在[官方网站](https://www.docker.com/)直接下载符合你操作系统的版本，安装即可。由于国内的网络下载Docker镜像实在是太慢，我们不妨使用国内的加速服务，教程可以参见：[https://yeasy.gitbooks.io/docker_practice/content/install/mirror.html](https://yeasy.gitbooks.io/docker_practice/content/install/mirror.html)

## Nginx
如果你还没有来得及看Docker的文档，当然也没关系，我们通过第一个服务，Nginx的搭建来边做边学。

首先，我们先拉取一个Nginx镜像，很简单：
```
> docker pull nginx
```
使用`docker images`查看一下我们刚刚拉取下来的镜像：

![Untitled Image](http://ooyc2y4k2.bkt.clouddn.com/VCBia)

**实例化**该镜像，我们把**实例化**的镜像称之为容器，镜像和容器的关系就好比类和实例的关系。

```
> docker run -p 80:80 nginx
```
Nginx容器启动之后，访问localhost，熟悉的欢迎页面出现了吧，不用我们下载，编译，安装，直接实例化然后启动就好了，多么让人幸福的一件事。

![Untitled Image](http://ooyc2y4k2.bkt.clouddn.com/wmvzy)

切回命终端，发现终端打印出了一堆log信息，我可不想整天盯着这些无趣的信息看，让Nginx进入守护运行吧。

```
> docker run -p 80:80 --name cool_nginx -d nginx
```
`-p`是端口参数，给上`-d`参数表示容器是守护程序会进入后台运行，`--name`则是重新给容器命名。成功之后，用`docker ps`来查看当前已经启动的容器。

![Untitled Image](http://ooyc2y4k2.bkt.clouddn.com/hsEm0)

现在我们进入这个容器，并修改这个Nginx默认网页。

```
> docker exec -it cool_nginx bash
> echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```
挂载一个本地目录到容器，就不用每次都进入到容器中修改代码了，当然的将`/Users/bruce/Documents/Workspace/Docker/src/`替换成你自己的文件目录。

```
> docker run -p 80:80 \
--name cool_nginx -d \
-v /Users/bruce/Documents/Workspace/Docker/src/:/usr/share/nginx/html/ \
nginx
```

# PHP
```
> docker pull php:7.0-fpm
```
使用7.0版本的PHP，你可以选择其他版本的PHP。
```
> docker run -d \
--name cool_php_fpm \
-v /Users/bruce/Documents/Workspace/Docker/src/:/usr/share/nginx/html/ \
php:7.0-fpm
```
使用`docker ps`可以看到，Nginx和FPM都已经启动了。

![Untitled Image](http://ooyc2y4k2.bkt.clouddn.com/3Z1Kc)

那么PHP如何和Nginx链接起来呢？很简单，Docker为我们做好了一切，只需要一个参数就可以将两个容器链接起来。
```
--link cool_php_fpm 
```
我们都知道，Nginx默认是不解析PHP文件的，所以还需要修改一下配置，不需要进入Nginx容器里去修改，我们在当前文件夹下，新建一个`default_nginx.conf`文件，写入如下内容：
```
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    access_log off;
    error_log  /var/log/nginx/error.log error;

    sendfile off;

    client_max_body_size 100m;

    location ~ \.php?$ {
        fastcgi_pass cool_php_fpm:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME /usr/share/nginx/html$fastcgi_script_name;
        fastcgi_intercept_errors off;
        fastcgi_buffer_size 16k;
        fastcgi_buffers 4 16k;
    }

    location ~ /\.ht {
        deny all;
    }
}
```
然后用`docker rm cool_nginx`命令删除掉原先的Nginx容器，挂载该配置文件重新实例化：
```
> docker run -p 80:80 \
--name cool_nginx -d \
-v /Users/bruce/Documents/Workspace/Docker/src/:/usr/share/nginx/html/ \
-v /Users/bruce/Documents/Workspace/Docker/default_nginx.conf:/etc/nginx/conf.d/default.conf:ro \
--link cool_php_fpm \
nginx
```
在本地`src/`目录下，新建一个`test.php`文件，写入熟悉的内容：
```
<?php
echo phpinfo();
```
Boom!!! PHP和Nginx就搭建好了，足够简单吧。
![phpinfo](http://ooyc2y4k2.bkt.clouddn.com/k2XcB)

## MySQL
和之前的安装一样，先拉取MySQL镜像：
```
> docker pull mysql
```
然后启动：
```
> docker run -d \
--name cool_mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
mysql
```
`-e`参数是给给定环境变量，这里我们设定MySQL的密码是`123456`。

链接到MySQL容器：
```
> docker exec -it cool_mysql bash
```
登陆MySQL:
```
> mysql -uroot -p123456
```
![MySQL](http://ooyc2y4k2.bkt.clouddn.com/d3Duk)

PHP如何链接到MySQL，相信你也知道了，对的还是使用`--link`参数，我们删除掉`cool_php_fpm`容器，重新构建
```
> docker run -d \
--name cool_php_fpm \
-v /Users/bruce/Documents/Workspace/Docker/src/:/usr/share/nginx/html/ \
--link cool_mysql \
php:7.0-fpm
```
链接之后，PHP容器和MySQL容器能够通信了，但是还是不够呢(坚持一下，最后一步了)，初始PHP是没有安装MySQL扩展的，安装扩张也极其容易，先进入到PHP容器：
```
> docker exec -it cool_php_fpm bash
```
然后用`docker-php-ext-install`命令安装：
```
> docker-php-ext-install mysqli
> exit
```
ok，我们重启PHP容器:
```
> docker restart cool_php_fpm
```
来测试一下，新建一个测试脚本：
```php
<?php

$servername = "cool_mysql";
$username = "root";
$password = "123456";

// Create connection
$conn = new mysqli($servername, $username, $password);

// Check connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
} 
echo "Connected successfully";
```
![test mysql success](http://ooyc2y4k2.bkt.clouddn.com/qsMdH)

## docker-compose
每次都用Dokcer命令实在是太过麻烦，引用`docker-compose`这个工具更方便构建容器，配置容器等工作。

好的，我们重新开始，使用下面两条命令让你忘掉一切：

```
> docker stop $(docker ps -a -q)
> docker rm $(docker ps -a -q)
```
我们新建一个名为`nginx`的目录，用来存放`nginx`容器相关的配置文件，把前面我们使用的Nginx配置文件`default_nginx.conf`移动到这里，并新建名为`Dockerfille`的文件，写入如下内容：
```
FROM nginx

COPY ./default_nginx.conf /etc/nginx/conf.d/default.conf
```
在**此文件夹**外新建`docker-compose.yml`的文件，并写入：
```
cool_nginx:
  build: ./nginx
  ports:
    - "80:80"
  volumes:
    # source
    - ./src/:/usr/share/nginx/html
```
想要重新构建Nginx容器就变得很简单：
```
> docker-compose build
```
启动Nginx容器也非常容易：
```
> docker-compose up -d
```
或者干脆两者结合起来：
```
> docker-compose up -d --build
```
将配置写入文件，不仅不容易出错，而且更加容易分发，谁也不想在终端上输入那么一长串的命令。

至于MySQL和PHP容器的构建，就不详细说明了，他们的构建都是一样的，全部的代码你可以在如下地址查阅：[https://github.com/blue7wings/lnmp_in_docker](https://github.com/blue7wings/lnmp_in_docker)。

## 后续
毕竟此教程是一个新手向的教程，不准确表达在所难免，如果对某些问题还是比较疑惑，可以留言询问，除此之外，我最为推荐的还是先去查阅官方文档，或者是去stackoverflow上去看一看。以下两个资源非常适合新手入门，希望所有朋友都能以此为入门，掌握这门优秀的技术。

- [Docker — 从入门到实践](https://yeasy.gitbooks.io/docker_practice/content/)
- [官方文档](https://docs.docker.com/)
