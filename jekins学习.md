docker 部署前后端



- 官方文档
  - https://www.jenkins.io/zh/doc/tutorials/build-a-java-app-with-maven/



- 启动命令

  - ```bash
    docker run --name jekins \
    -u root -p 8080:8080 \
    -e JAVA_OPTS="-Dhudson.plugins.git.GitSCM.ALLOW_LOCAL_CHECKOUT=true" 
    -v jenkins-data:/var/jenkins_home \ 
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v "$HOME":/home jenkinsci/blueocean
    ```

  - 注意这里的-v `$HOME":/home`做了映射，因此在【绑定本机`repository`】时应该使用`/home/...`

  - `--rm` 的意思是容器运行结束后【直接删除容器】

  

```bash
$ docker run -t -d -u 0:0 -v /root/.m2:/root/.m2 -w /var/jenkins_home/workspace/test --volumes-from
......
java.io.IOException: Failed to run image 'maven:3-alpine'. Error: docker: Error response from daemon: Mounts denied:
The path /root/.m2 is not shared from the host and is not known to Docker.

# 这里的/root/.m2:/root/.m2并没有做过映射，因此需要使用本机路径或者做一个映射
You can configure shared paths from Docker -> Preferences... -> Resources -> File Sharing.
```

```bash
# maven可能会因为版本老旧报错，docker pull新版本即可
```



根据教程 

https://blog.csdn.net/Wenda_/article/details/102956134

https://blog.csdn.net/ITshu/article/details/103030308 

与一些探索实现github webhook与jekins的联系，自动监控提交并且测试部署

- 问题

jekins部署在本地8080端口，由于是私有网络，只是在本地同网络下postman测试行得通，github webhook报错

```bash
We couldn’t deliver this payload: failed to connect to host
```

根据github issues，使用ngrok，github连接上了jekins，但是报错

```bash
Failed to add GitHub webhook for GitHubRepositoryName[host=github.com,username=WhiteStart,repository=marathon]
org.kohsuke.github.HttpException: {"message":"Validation Failed","errors":[{"resource":"Hook","code":"custom","message":"Sorry, the URL host 127.0.0.1 is not supported because it isn't reachable over the public 
```

通过文章 https://webhookrelay.com/blog/2017/11/23/github-jenkins-guide/#Step-6-Setting-up-Webhook-Relay-agent 【==Receive Github webhooks on Jenkins without public IP==】使用Webhook Relay解决



key

541210ec-4dbd-41f2-81f3-59aa486774cd

sercet

lW3UaJzZ37kP



注意对Jekinsfile内容的更新，当jekinsfile中的docker命令出错时，jekins客户端中会显示success，但是docker中可能不会有当前容器的信息。
