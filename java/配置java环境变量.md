```bash
1.oracle archive 下载tar.gz文件
2.tar -zxvf {} -C usr/local
3.cd /usr/local
4.mv {} jdk
5.vim /etc/profile
6.# Java
export JAVA_HOME=/usr/local/jdk
export PATH=$PATH:$JAVA_HOME/bin
7.source /etc/profile
```

