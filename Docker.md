# Docker安装

```shell
sudo apt-get remove docker docker-engine docker.io containerd runc # 卸载旧版本

sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

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

VERSION_STRING=5:24.0.0-1~ubuntu.22.04~jammy
sudo apt-get install docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-buildx-plugin docker-compose-plugin
```

# Docker删除

- 查看docker是否卸载干净

```javascript
dpkg -l | grep docker

# 删除无用的相关的配置文件
apt-get autoremove docker docker-ce docker-engine  docker.io  containerd runc
dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P 
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



# Docker实践

编译Dockerfile，生成image

```
docker build -t demo:v1.0 .
```

## nginx

```shell
docker run -d -p 1234:80 \
-v /root/data/html:/usr/share/nginx/html:ro \
-v /root/data/conf/nginx.conf:/etc/nginx/nginx.conf \
--name nginx_test \
nginx

docker cp 431d82f30829:/etc/nginx/nginx.conf /root/data/conf/nginx.conf
```

## redis

```console
docker run -v /root/data/redis/redis.conf:/etc/redis/redis.conf \ 
--name myredis \
-p 6379:6379 \
-d redis redis-server /etc/redis/redis.conf
```

## mysql

```bash
# 启动一个mysql 密码设置为hmz990203 并通过数据卷挂载
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=hmz990203 -v /temp/mysql/conf/test.cnf:/etc/mysql/conf.d/test.cnf -v /temp/mysql/data:/var/lib/mysql -d mysql

# 运行进入mysql命令行
docker exec -it my-mysql mysql -uroot -p
```

## ELK

### 1.elasticsearch

es与kibana，参考：https://blog.csdn.net/Box_clf/article/details/130570821

- 启动

```bash
docker run -d --name es -e "discovery.type=single-node" \
-e "ES_JAVA_OPTS=-Xms2048m -Xmx2048m" \
-v /Users/huangminzhi/Desktop/elk/es/es-data:/usr/share/elasticsearch/data \
-v /Users/huangminzhi/Desktop/elk/es/es-plugins:/usr/share/elasticsearch/plugins \
-v /Users/huangminzhi/Desktop/elk/es/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
--privileged=true --network es-net -p 9200:9200 -p 9300:9300 \
elasticsearch:8.7.0

```

- elasicsearch.yml

```yml
network.host: 0.0.0.0   
#外网可访问
network.bind_host: 0.0.0.0  

http.cors.enabled: true
http.cors.allow-origin: "*"
# 这条配置表示开启xpack认证机制 spring boot连接使用
xpack.security.enabled: true
# 内部网络环境中禁用了 Elasticsearch 与 Logstash 之间的传输层安全性。这可能在某些生产环境中不推荐使用
xpack.security.transport.ssl.enabled: false
# 许可证，可能与logstash有关
xpack.license.self_generated.type: basic 
```

- 初始化密码

```bash
bin/elasticsearch-setup-passwords interactive
# 默认都设置了abcdef[密码必须是字符串]

#增加授权：
#superuser能正常打开es的9200端口，kibana_system配置后才可以正常对接kb和es
bin/elasticsearch-users useradd kibana_user
bin/elasticsearch-users roles -a superuser kibana_user
bin/elasticsearch-users roles -a kibana_system kibana_user

#移除授权：
bin/elasticsearch-users roles -r kibana_admin kibana_user

#查看授权：
bin/elasticsearch-users roles -v kibana_user
```

- 导入ik分词器

```bash
# 查询volumes目录
docker volume inspect es-plugins

[
    {
        "CreatedAt": "2023-06-25T01:36:23Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/es-plugins/_data",
        "Name": "es-plugins",
        "Options": null,
        "Scope": "local"
    }
]
# 创建ik文件夹
mkdir ik
# 将ik分词器cp过去，在解压并重启es
docker cp elasticsearch-analysis-ik-8.7.0.zip 8d0a17bfa248:/usr/share/elasticsearch/plugins/ik
```

### 2.kibana

- 启动

```bash
# 根据上述 es 配置启动 kibana
docker run -d --name kibana \
-e "ELASTICSEARCH_HOSTS=http://es:9200" \
-e "ELASTICSEARCH_USERNAME=kibana_user" \
-e "ELASTICSEARCH_PASSWORD=abcdef" \
--network es-net -p 5601:5601 kibana:8.7.0
```

```bash
# 账号不可以是elastic这个默认的主账号
value of "elastic" is forbidden. This is a superuser account that cannot write to system indices that Kibana needs to function. Use a service account token instead. Learn more: https://www.elastic.co/guide/en/elasticsearch/reference/8.0/service-accounts.html
// ......
```



#### 失败之处

根据上述链接并没有运行成功

```bash
# es 重置密码
elasticsearch-reset-password -u elastic

# 创建serveice token
curl -u elastic:YB1weIMGrzUUH3CJZyWE -X POST "localhost:9200/_security/service/elastic/kibana/credential/token/kibana_token?pretty"

# 验证service token
curl -H "Authorization: Bearer AAEAAWVsYXN0aWMva2liYW5hL2tpYmFuYV90b2tlbjpQc19ZalZCRFR0MjZpbTZMeVF1SEhn" http://localhost:9200/_cluster/health
```



### 3.logstash

- <font color=red>logstash是可以日志采集，但是资源消耗比较大，其主要功能是日志的过滤以及格式化的输出</font>

- <font color=red>filebeat是代替其采集功能，消耗的资源小，哪台机器需要日志采集就部署filebeat</font>



logstash与filebeat，参考：https://juejin.cn/post/7025987191432019982

- 为logstash配置权限

```bash
bin/elasticsearch-users useradd logstash_writer
bin/elasticsearch-users roles -a superuser logstash_writer
bin/elasticsearch-users roles -a logstash_system logstash_writer

#查看授权：
bin/elasticsearch-users roles -v logstash_writer

# 验证是否添加成功
curl -u logstash_writer:abcdef -X GET http://localhost:9200/
```

- 启动

```bash
docker run -it -d -p 5044:5044 \
-v /Users/huangminzhi/Desktop/elk/log/logstash:/var/log/logstash \
-v /Users/huangminzhi/Desktop/elk/logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml \
-v /Users/huangminzhi/Desktop/elk/logstash/conf.d/:/usr/share/logstash/conf.d/ \
--name logstash --network es-net logstash:8.7.0
```

【注】：==对于.conf要严格遵守缩进==

- conf.d下的test.conf

```conf
input {
  beats {
    port => 5044
  }
}

filter {
  if [message] =~ /test_error2/ {
    drop { }
  }
}

output {
  elasticsearch {
    hosts => ["http://es:9200"]
    index => "test-%{+YYYY.MM.dd}"
    user => "logstash_writer"
    password => "abcdef"
  }
}

```

- config下的logstash.yml

```yml
http.host: '0.0.0.0'
xpack.monitoring.enabled: true
xpack.monitoring.elasticsearch.hosts: ['http://es:9200']
# 以下两行缺失会出现如下报错
xpack.monitoring.elasticsearch.username: 'logstash_writer'
xpack.monitoring.elasticsearch.password: 'abcdef'

path.config: /usr/share/logstash/conf.d/*.conf
path.logs: /var/log/logstash
```

```bash
Unable to retrieve license information from license server {:message=>"Got response code '401' contacting Elasticsearch at URL 'http://es:9200/_xpack'"}
[2023-06-26T05:59:46,601][WARN ][logstash.licensechecker.licensereader] Attempted to resurrect connection to dead ES instance, but got an error {:url=>"http://es:9200/", :exception=>LogStash::Outputs::ElasticSearch::HttpClient::Pool::BadResponseCodeError, :message=>"Got response code '401' contacting Elasticsearch at URL 'http://es:9200/'"}
```



### 4.filebeat

测试能否监听到nginx的日志

- 启动nginx

```bash
docker run --name nginx-test -p 8888:80  -v /Users/huangminzhi/Desktop/elk/nginx:/var/log/nginx  -d nginx
```

- 启动filebeat

```bash
docker run -d --name filebeat --user=root --network es-net \
-v /Users/huangminzhi/Desktop/elk/nginx/:/var/log/nginx/ \
-v /Users/huangminzhi/Desktop/elk/log/:/var/log/marathon/ \
-v /Users/huangminzhi/Desktop/elk/logstash/filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro \
-v /var/lib/docker/containers:/var/lib/docker/containers \
-v /var/run/docker.sock:/var/run/docker.sock:ro \
elastic/filebeat:8.7.0
```

```yml
filebeat.config:
  modules:
    path: ${path.config}/modules.d/*.yml
    reload.enabled: false

filebeat.autodiscover:
  providers:
    - type: docker
      hints.enabled: true

processors:
  - add_cloud_metadata: ~

# 直接输出到elasticsearch
# 如果不需要对日志进行过滤和格式化，则可以直接输出到elasticsearch

# output.elasticsearch:
#   hosts: '${ELASTICSEARCH_HOSTS:elasticsearch:9200}'
#   username: '${ELASTICSEARCH_USERNAME:}'
#   password: '${ELASTICSEARCH_PASSWORD:}'

filebeat.inputs:
  - type: log
    enabled: true
    paths:
      # filebeat监听【容器】中该路径下的日志
      - /var/log/nginx/*.log
    tags: ['nginx']
output.logstash:
  hosts: ['logstash:5044']
  
# 在启动命令中
# -v /Users/huangminzhi/Desktop/elk/nginx/:/var/log/nginx/
# 是为了让filebeat可以访问到该目录
```



