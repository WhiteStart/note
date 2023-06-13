# Docker安装

```shell
sudo apt-get remove docker docker-engine docker.io containerd runc # 卸载旧版本
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"

sudo add-apt-repository "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu $(lsb_release -cs) stable"

sudo apt-get update

# 查看可安装的版本
apt-cache madison docker-ce

# 安装指定版本docker
sudo apt-get install docker-ce=5:20.10.7~3-0~ubuntu-bionic

# 安装最新版
sudo apt install docker.io

# 设置开机启动docker，并立即启动
systemctl enable docker --now

# 启动docker
sudo systemctl start docker
```

# Docker删除

- 删除docker及安装时自动安装的所有包

```
apt-get autoremove docker docker-ce docker-engine  docker.io  containerd runc
```

- 查看docker是否卸载干净

```javascript
dpkg -l | grep docker

dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P # 删除无用的相关的配置文件

apt-get autoremove docker-ce-*
  
rm -rf /etc/systemd/system/docker.service.d

rm -rf /var/lib/docker
```



# Docker常用命令

## 1.帮助命令

```shell
# 显示版本信息
docker version

# 显示系统信息，，
docker info

# 帮助命令
docker {} --help
```



## 2.镜像命令

### 1.docker images(查看本机镜像)

```shell
# 可选项
-a, --all     列出所有的镜像
-q, --quiet 	只显示镜像的id
```

### 2.docker search(搜索镜像)

```shell
docker search mysql

# 可选项
--filter=stars=3000 搜索stars>3000的
```

### 3.docker pull(下载镜像)

```shell
# 最新版
docker pull mysql

# 指定版本下载
docker pull mysql:8.0
```

#### 4.docker rmi(删除镜像)

```shell
# 删除若干个
docker rmi -f {IMAGE ID} {}

# 删除全部
docker rmi -f $(docker images -aq)
```



## 3.容器命令

### 1.新建容器并启动

```shell
docker run [options] image
# 容器名称
--name="Name" 

# 后台方式运行(必须要有一个前台进程)
-d

# 交互式方式运行，进入容器查看内容
-it

# 指定容器端口
-p
	1.-p ip:主机端口:容器端口
	2.-p 主机端口:容器端口(常用)
	3.-p 容器端口
# 随机指定端口
-P 

## 测试
root@abc:~# docker run -it ubuntu /bin/bash
root@9317505abdab:/# 
```

### 2.列出容器

```shell
# 正在运行的容器
docker ps
# 所有容器
docker ps -a
# 仅列出id
docker ps -aq
```

### 3.退出容器

```shell
# 直接退出容器并停止
exit
# 退出并后台运行
ctrl+q+p
```

### 4.删除容器

```shell
# 删除指定容器
docker rm {id}
# 删除全部 / -f(强制)
docker rm -f $(docker ps -aq)
```

### 5.启动/停止容器

```
docker start {id}

docker restart {id}

docker stop {id}

docker kill {id}
```

### 6.查看容器信息

```shell
docker inspect {id}
```

### 7. 后台方式进入容器

- docker exec 进入容器后开启一个新的终端，可以在里面操作(常用)

==docker exec -it {id} /bin/bash==

### 8.容器提交至本地仓库

- docker commit -a "hmz" -m "first commit" f05eea7b28a8 nginx_hmz

### 9.镜像传输

- 压缩成本地镜像(离线安装)
  - docker save -o test.tar nginx 
  - scp传输
  - 加载 docker load -i test.tar
- 传到docker hub上
  - docker login
    - 输入账号密码
  - ==docker tag nginx_test whitestart/test:v1.0==
    - 将旧的镜像名改成==仓库要求==的新名字，打成标签
  - ==docker push whitestart/test:v1.0==
    - 上传

### 10.挂载数据到数据卷修改

docker run -d --name "nginx_hmz" -p 1234:80 \

==-v /data/html:/usr/share/nginx/html:ro==

- 将容器内/usr/share/nginx/html挂载到外部/data/html修改
  - ro 表示容器内只读 
  - rw 表示容器内读写


### 11.查询日志

```shell
docker logs 容器名/id
```

```shell
docker run -d -p 1234:80 \
-v /root/data/html:/usr/share/nginx/html:ro \
-v /root/data/conf/nginx.conf:/etc/nginx/nginx.conf \
--name nginx_test \
nginx

docker cp 431d82f30829:/etc/nginx/nginx.conf /root/data/conf/nginx.conf
```

```console
docker run -v /root/data/redis/redis.conf:/etc/redis/redis.conf \ 
--name myredis \
-p 6379:6379 \
-d redis redis-server /etc/redis/redis.conf
```



```
docker build -t demo:v1.0 .
```





```bash
# 启动一个mysql 密码设置为hmz990203 并通过数据卷挂载
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=hmz990203 -v /temp/mysql/conf/test.cnf:/etc/mysql/conf.d/test.cnf -v /temp/mysql/data:/var/lib/mysql -d mysql

# 运行进入mysql命令行
docker exec -it my-mysql mysql -uroot -p
```



```bash
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=hmz990203 -v /Users/huangminzhi/Desktop/cloud/temp/mysql/conf/test.cnf:/etc/mysql/conf.d/test.cnf -v /Users/huangminzhi/Desktop/cloud/temp/mysql/data:/var/lib/mysql -d mysql
```

```
docker-compose up -d
```

```
docker run -d --name es -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" -e "discovery.type=single-node" -v es-data:/usr/share/elasticsearch/data -v es-plugins:/usr/share/elasticsearch/plugins --privileged --network es-net -p 9200:9200 -p 9300:9300 elasticsearch:7.17.10

docker run -d --name kibana -e ELASTICSEARCH_HOSTS=http://es:9200 --network=es-net -p 5601:5601 kibana

```

