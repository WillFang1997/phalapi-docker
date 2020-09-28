# 使用Docker快速部署你的Phalapi项目

## 为什么要用docker

<img src=http://cd7.yesapi.net/89E670FD80BA98E7F7D7E81688123F32_20200928215312_dee6612f9e5b862c37cd816c3222ce81.jpeg?>

docker作为一种新兴的虚拟化技术，相较于传统的部署方式，使用docker去部署我们的项目，可以有更多的好处，如

### 更快的部署，启动时间
传统去部署我们的phalapi项目或phalapi-pro项目，需要安装nginx，php，fpm，mysql等等环境，php的拓展，nginx的配置，fpm的配置等等问题都可以
花费很多时间，即使自己亲手部署过很多次，隔一小段时间后重新去部署一个新环境也会出现其他问题，使用docker可以大大缩短我们的部署时间，启动时间，让
我们有更多的时间，精力去愉快的coding(摸鱼)！！！

### 一致的环境，消除不同环境的隐藏问题
在开发的过程中，常常会听到这么一句话：这个在我的电脑是好好的呀，怎么到你这就有问题！由于开发环境，测试环境，生产环境的不一致，导致有一些bug并未在
开发的过程中被发现。Docker镜像提供了除内核外完整的运行时环境，确保了应用运行环境一致性。

### 持续构建，部署，迁移
对开发和运维(DevOps)人员来说，最希望的就是一次创建或配置，可以在任意地方正常运行。Docker 确保了执行环境的一致性，使得应用的迁移更加容易。Docker
可以在很多平台上运行，无论是物理机、虚拟机、公有云、私有云，甚至是我们自己不同系统的电脑，其运行结果是一致的。 因此用户可以很轻易的将在一个平台上运行
的应用，迁移到另一个平台上，而不用担心运行 环境的变化导致应用无法正常运行的情况。

## 正式开始

接下来，废话不多说，正式开始！首先开始准备我们的环境，由于我们部署的需要nginx和php环境，这里使用docker-compose去编排我们的镜像

### 安装docker
```shell script
sudo yum install docker
# 建立 docker 组：
sudo groupadd docker
# 将当前用户加入docker组
sudo usermod -aG docker $USER
# 启动docker
sudo service docker start
sudo systemctl enable docker
```

### 安装docker-compose
```shell script
sudo curl -L https://github.com/docker/compose/releases/download/1.23.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### 准备我们的工作目录
```shell script
# 假设我们的工作目录是：/home/app
export $DOCKERPATH =/home/app
# 将项目clone，并移入/home/app目录下
git clone https://github.com/shuxnhs/phalapi-docker.git
```

## 运行我们的项目

1. 首先提供我们的docker-compose.yml,可以将本项目clone到/home目录
```yaml
version: '3'
services:
  nginx:
    image: nginx
    ports:
      - "8000:80"
    depends_on:
      - phalapi
    volumes:
      - "$DOCKERPATH/app/nginx/conf.d:/etc/nginx/conf.d"
      - "$DOCKERPATH/app/nginx/nginx.conf:/usr/local/nginx/conf/nginx.conf"
      - "$DOCKERPATH/app/nginx/html:/usr/share/nginx/html"
      - "$DOCKERPATH/app/nginx/log:/var/log/nginx"
    networks:
      - phalapi_net
    container_name: "phalapi_nginx"
  phalapi:
    image: shuxnhs/phalapi:latest
    ports: ["9000"]
    volumes:
      - "$DOCKERPATH/app/nginx/html/phalapi:/var/www/html/phalapi"
    networks:
      - phalapi_net
    container_name: "phalapi"
networks:
  phalapi_net:
```

2.介绍一下docker-compose.yml的命令作用
+ nginx中，将我们的配置，日志挂载数据卷，同时将我们的phalapi项目放到/app/nginx/html目录下挂载进容器
+ phalapi中是我们的php环境，这里最新的镜像使用的是php-7.1的环境，安装了常用的gd，pdo_mysql，mcrypt扩展，如果有需要php5的环境后续再更新的镜像

3.开始运动我们的项目
```shell script
docker-compose up -d
```

4. 打开我们的浏览器：127.0.0.1:8000即可以看到我们的项目首页：
<img src=http://cd7.yesapi.net/89E670FD80BA98E7F7D7E81688123F32_20200928232349_2d6400e6eb7f384359b5c06bf7610f52.png?>


完美，如果感觉有帮助欢迎来github给个小星星✨，后面再来讲讲结合容器中计划任务相关的东西。
