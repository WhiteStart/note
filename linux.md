# 文件权限显示

```
-  ---  ---  ---
d	 rwx  rw-  r--

1.代表文件类型
  - 文件
  d 文件夹
  l 表示链接

2.当前用户具有该文件的权限
	r 读(read)
	w 写(write)
	x 执行(excute)
	
3.当前组内其他用户具有该文件的权限
	r
	w
	x

4.其他组的用户具有该文件的权限
	r
	w
	x
```

# chmod命令

| u    | user   | 文件所有者       |
| ---- | ------ | ---------------- |
| g    | group  | 文件所有者所在组 |
| o    | others | 所有其他用户     |
| a    | all    | 所有用户         |

```markdown
# u增加读权限
chmod u + r file.txt

# u和g取消执行权限
chmod ug - x file.txt
```

```
chmod 命令也可以使用八进制来指定权限

chmod (user 文件所有者)(group 用户组)(other 其他用户) {file_name}
```

| #    | 权限       | rwx  | 二进制 |
| ---- | ---------- | ---- | ------ |
| 7    | 读+写+执行 | rwx  | 111    |
| ...  |            |      |        |
| 4    | 只读       | r--  | 100    |
| ...  |            |      |        |
| 1    | 只执行     | --x  | 001    |
| 0    | 无         | ---  | 000    |

```
chmod 751 file.txt

文件所有者拥有7，即rwx
组内用户拥有5， 即r-x
其他用户拥有1，即--x
```



# netstat

```bash
# 显示当前系统上所有TCP和UDP网络连接的详细信息，并列出与每个连接相关的进程信息。
netstat -nutlp
```

![截屏2023-07-24 下午5.49.27](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-07-24 下午5.49.27.png)



# iptables

```bash
iptables -vnL -t nat --line | grep 2888
```

- `-v`: 显示详细信息，包括数据包和字节计数。
- `-n`: 以数字形式显示IP地址和端口号，而不进行DNS解析。
- `-L`: 列出规则。在没有指定规则链名称时，将显示所有规则链的内容。
- `-t nat`: 指定要查看的表为"nat"表，该表用于配置网络地址转换（NAT,Network Address Translation）规则。
- `--line`: 显示规则的行号。



# tar

```bash
wget http://ftp.gnu.org/gnu/gcc/gcc-11.2.0/gcc-11.2.0.tar.gz
tar -zxvf gcc-11.2.0.tar.gz
```

- `-x`: 解压缩一个tar归档文件。
- `-z`: 使用gzip压缩/解压缩。当使用`-z`选项时，tar会通过gzip来处理归档文件。
- `-c`: 创建一个tar归档文件。
- `-f`: 指定归档文件的名称。在此处，应该紧接着`-f`选项后面提供归档文件的名称。



# dpkg

```bash
## 解压软件包
dpkg -i package.deb

# 卸载软件包
dpkg -r package

# 移除软件包（包括配置文件）
dpkg -P package

## 更新软件包
dpkg --configure -a
```



```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt update
sudo apt install gcc-9
sudo mv /usr/bin/gcc /usr/bin/gcc_backup
sudo ln -s /usr/bin/gcc-9 /usr/bin/gcc
```

```bash
tar -zxvf redis-7.0.12.tar.gz
cd redis-7.0.12/src
make install

mkdir -p /usr/local/redis/etc
mkdir -p /usr/local/redis/bin
mkdir -p /usr/local/redis-cluster

mv mkreleasehdr.sh redis-benchmark redis-check-aof redis-check-rdb redis-cli redis-server /usr/local/redis/bin
cd ..
cp redis.conf /usr/local/redis/etc
cp redis.conf /usr/local/redis-cluster

cd /usr/local/redis-cluster
mkdir 6001
mkdir 6002


# 将redis中的daemonize改为yes，再启动 
/usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf
# 进入redis客户端
/usr/local/redis/bin/redis-cli


%s/6001/6002/g

sudo apt install ruby

/usr/local/redis/bin/redis-server /usr/local/redis-cluster/6001/redis.conf 

redis-cli --cluster create 121.43.180.239:6001 121.43.180.239:6002 47.96.65.118:6003 47.96.65.118:6004 47.99.68.19:6005 47.99.68.19:6006 --cluster-replicas 1 -a yfkn1#Z#1d

cluster info
cluster nodes

yfkn1#Z#1d
```

```
redis-cli --cluster create 172.16.34.23:6001 172.16.34.23:6002 172.16.34.24:6003 172.16.34.24:6004 172.16.34.29:6005 172.16.34.29:6006 --cluster-replicas 1 -a yfkn1#Z#1d
```

```
redis-cli -h <redis-host> -p <redis-port> -a <redis-password>
redis-cli -h 10.22.191.78 -p 6001 -a yfkn1#Z#1d
redis-cli -h 172.16.34.23 -p 6001 -a yfkn1#Z#1d
```



```
docker run -d --name kibana \
-e "ELASTICSEARCH_HOSTS=http://10.0.1.102:9200" \
--network zookeeper-kafka -p 5601:5601 kibana:7.6.2
```

```
curl -XGET -u elasticuser:yourpassword http://localhost:9200/_cluster/health\?pretty | grep unassigned_shards
```

```
curl -XGET localhost:9200/_cat/shards?h=index,shard,prirep,state,unassigned.reason| grep UNASSIGNED
```

```
curl -XGET -u elasticuser:yourpassword http://localhost:9200/_cluster/health
```

```
docker run -d --name es -e "discovery.type=single-node" \
-e "ES_JAVA_OPTS=-Xms2048m -Xmx2048m" \
-v /jinhetech_workspace/ELK/es/es-data:/usr/share/elasticsearch/data \
-v /jinhetech_workspace/ELK/es/es-plugins:/usr/share/elasticsearch/plugins \
-v /jinhetech_workspace/ELK/es/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
--privileged=true --network es-net -p 9201:9200 -p 9301:9300 \
elasticsearch:8.7.0

docker run -d --name kibana \
-e "ELASTICSEARCH_HOSTS=http://es:9200" \
-e "ELASTICSEARCH_USERNAME=kibana_user" \
-e "ELASTICSEARCH_PASSWORD=jinhetech" \
--network es-net -p 5601:5601 kibana:8.7.0

docker run -it -d -p 5044:5044 \
-v /jinhetech_workspace/elk/log/logstash:/var/log/logstash \
-v /jinhetech_workspace/elk/logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml \
-v /jinhetech_workspace/elk/logstash/conf.d/:/usr/share/logstash/conf.d/ \
--name logstash --network es-net logstash:8.7.0

docker run -d --network es-net --name filebeat --user=root \
-v /jinhetech_workspace/logs:/var/lib/docker/volumes/xfile:ro \
-v /jinhetech_workspace/elk/logstash/filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro \
-v /var/lib/docker/containers:/var/lib/docker/containers \
-v /var/run/docker.sock:/var/run/docker.sock:ro \
elastic/filebeat:8.7.0
```



[sun@hadoop102 kafka]$ ./kafka-producer-perf-test.sh  --topic my-topic --record-size 100 --num-records 100000 --throughput -1 --producer-props bootstrap.servers=kafka1:9092,kafka2:9092,kafka3:9092

```
kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic abc
```

```
./kafka-console-producer.sh --broker-list localhost:9092 --topic my-topic --producer.config ../config/producer.properties
```

```
kafka-console-producer.sh --broker-list localhost:9092 --topic my-topic --producer-property security.protocol=SASL_PLAINTEXT --producer-property sasl.mechanism=SCRAM-SHA-256 --producer-property Djava.security.auth.login.config=../config/kafka_jaas.conf
```

KAFKA_CFG_LISTENER_NAME_INTERNAL_SASL_JAAS_CONFIG

```text
listener.name.sasl_ssl.scram-sha-256.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required \
    username="admin" \
    password="admin-secret";
```

```
kafka-console-producer.sh --producer.config /opt/bitnami/kafka/config/producer.properties --bootstrap-server kafka1:9092 --topic test
```



压测数据

- 生产者

```
./kafka-producer-perf-test.sh  --topic my-topic --record-size 100 --num-records 100000 --throughput -1 --producer-props bootstrap.servers=kafka1:9092,kafka2:9092,kafka3:9092
```

```
100000 records sent, 28145.229384 records/sec (2.68 MB/sec), 919.54 ms avg latency, 1323.00 ms max latency, 904 ms 50th, 1047 ms 95th, 1084 ms 99th, 1120 ms 99.9th.
```

- 消费者

```
./kafka-consumer-perf-test.sh --broker-list kafka1:9092,kafka2:9092,kafka3:9092 --topic test --fetch-size 10000 --messages 10000000 --threads 1
```



**分区数=Tt /min（Tp，Tc）**

Kafka机器数量（经验公式）=2 *（峰值生产速度*副本数/100）+1



kafka-consumer-perf-test.sh --zookeeper 10.22.191.78:2181 --topic test --fetch-size 10000 --messages 10000000 --threads 1



```
redis-cli -h 218.94.154.74 -p 6879 -a jinhetech

```





```
error_exit ()
{
    echo "ERROR: $1 !!"
    exit 1
}

[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=$HOME/jdk/java
[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/usr/java
[ ! -e "$JAVA_HOME/bin/java" ] && unset JAVA_HOME

if [ -z "$JAVA_HOME" ]; then
  if [ "Darwin" = "$(uname -s)" ]; then

    if [ -x '/usr/libexec/java_home' ] ; then
      export JAVA_HOME=`/usr/libexec/java_home`

    elif [ -d "/System/Library/Frameworks/JavaVM.framework/Versions/CurrentJDK/Home" ]; then
      export JAVA_HOME="/System/Library/Frameworks/JavaVM.framework/Versions/CurrentJDK/Home"
    fi
  else
    JAVA_PATH=`dirname $(readlink -f $(which javac))`
    if [ "x$JAVA_PATH" != "x" ]; then
      export JAVA_HOME=`dirname $JAVA_PATH 2>/dev/null`
    fi
  fi
  if [ -z "$JAVA_HOME" ]; then
        error_exit "Please set the JAVA_HOME variable in your environment, We need java(x64)! jdk8 or later is better!"
  fi
fi


export WEB_SERVER="ks-km"
export JAVA_HOME
export JAVA="$JAVA_HOME/bin/java"
export BASE_DIR=`cd $(dirname $0)/..; pwd`
export CUSTOM_SEARCH_LOCATIONS=file:${BASE_DIR}/conf/


#===========================================================================================
# JVM Configuration
#===========================================================================================

JAVA_OPT="${JAVA_OPT} -server -Xms2g -Xmx2g -Xmn1g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
JAVA_OPT="${JAVA_OPT} -XX:-OmitStackTraceInFastThrow -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=${BASE_DIR}/logs/java_heapdump.hprof"

## jdk版本高的情况 有些 参数废弃了
JAVA_MAJOR_VERSION=$($JAVA -version 2>&1 | sed -E -n 's/.* version "([0-9]*).*$/\1/p')
if [[ "$JAVA_MAJOR_VERSION" -ge "9" ]] ; then
  JAVA_OPT="${JAVA_OPT} -Xlog:gc*:file=${BASE_DIR}/logs/km_gc.log:time,tags:filecount=10,filesize=102400"
else
  JAVA_OPT="${JAVA_OPT} -Djava.ext.dirs=${JAVA_HOME}/jre/lib/ext:${JAVA_HOME}/lib/ext"
  JAVA_OPT="${JAVA_OPT} -Xloggc:${BASE_DIR}/logs/km_gc.log -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=100M"

fi

JAVA_OPT="${JAVA_OPT} -jar ${BASE_DIR}/libs/${WEB_SERVER}.jar"
JAVA_OPT="${JAVA_OPT} --spring.config.additional-location=${CUSTOM_SEARCH_LOCATIONS}"
JAVA_OPT="${JAVA_OPT} --logging.config=${BASE_DIR}/conf/logback-spring.xml"
JAVA_OPT="${JAVA_OPT} --server.max-http-header-size=524288"



if [ ! -d "${BASE_DIR}/logs" ]; then
  mkdir ${BASE_DIR}/logs
fi

echo "$JAVA ${JAVA_OPT}"

# check the start.out log output file
if [ ! -f "${BASE_DIR}/logs/start.out" ]; then
  touch "${BASE_DIR}/logs/start.out"
fi

# start
echo -e "---- 启动脚本 ------\n $JAVA ${JAVA_OPT}" > ${BASE_DIR}/logs/start.out 2>&1 &


nohup $JAVA ${JAVA_OPT} >> ${BASE_DIR}/logs/start.out 2>&1 &

echo "${WEB_SERVER} is starting，you can check the ${BASE_DIR}/logs/start.out"
```

```
update ks_km_physical_cluster set zk_properties='{ "openSecure": true, "otherProps": { "zookeeper.sasl.clientconfig": "KafkaClient" } }' where id=1;
```



```

    docker run -d --name kafka-exporter1 --network zookeeper-kafka -p 9308:9308 docker.io/bitnami/kafka-exporter --kafka.server=kafka1:9092 --kafka.version=3.4.1 --sasl.enabled --sasl.username=user --sasl.password=GiAszivMBB --sasl.mechanism=scram-sha256
    docker run -d --name kafka-exporter2 --network zookeeper-kafka -p 9308:9308 docker.io/bitnami/kafka-exporter --kafka.server=kafka2:9092 --kafka.version=3.4.1 --sasl.enabled --sasl.username=user --sasl.password=GiAszivMBB --sasl.mechanism=scram-sha256
    docker run -d --name kafka-exporter3 --network zookeeper-kafka -p 9308:9308 docker.io/bitnami/kafka-exporter --kafka.server=kafka3:9092 --kafka.version=3.4.1 --sasl.enabled --sasl.username=user --sasl.password=GiAszivMBB --sasl.mechanism=scram-sha256
```



```
docker run -d --network es-net --name filebeat --user=root \
-v /jinhetech_workspace/logs:/var/lib/docker/volumes/xfiles:ro \
-v /jinhetech_workspace/filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro \
-v /var/lib/docker/containers:/var/lib/docker/containers \
-v /var/run/docker.sock:/var/run/docker.sock:ro \
elastic/filebeat:8.7.0
```

