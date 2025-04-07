| 步骤 | 描述                                   |
|------|----------------------------------------|
| 1    | [动态构建 PV 和 PVC](1.创建动态构建PV，PVC/动态构建Pv,pvc.md)                     |
| 2    | [安装 GitLab](2.gitlab安装/gitlab安装.md)                            |
| 3    | [将 Git 关联到 GitLab 并推送项目](3.使用git推送项目到gitlab/使用git推送项目到gitlab.md)        |
| 4    | [安装 Harbor](4.Harbor安装/Harbor安装.md)                           |
| 5    | [安装 SonarQube](5.Sonarqube安装/Sonarqube安装.md)                         |
| 6    |➡️ [安装 Jenkins](6.jenlinks安装/jenlinks安装.md)                           |
| 7    | [连接各个中间件形成 DevOps](7.连接各个中间件形成devops/连接中间件形成Devops.md)         |
| 8    | [创建流水线任务](8.创建pipe流水线/创建流水线任务.md)    

## 一、整合Jenkins镜像
为了之后直接在 jenkins master的pod里构建jenkins任务，我们需要maven环境和sonarqube-cli命令行工具 
所以我们直接把maven和sonarqube-cli整合在jenkins的镜像里，之后部署Jenkins-deployment.yaml里直接使用该镜像来部署Jenkins到K8s

⭐可以直接在Dockerfile文件夹里用下载好的直接构建（sonar-scanner太大上传不上来，请用下面命令自行下载）  

下面是流程:
### 1.首先下载要添加到基础镜像中的组件
wget https://dlcdn.apache.org/maven/maven-3/3.9.9/binaries/apache-maven-3.9.9-bin.tar.gz

wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.8.0.2856-linux.zip

### 2.创建Dockerfile
`vim dockerfile`
```yaml
FROM jenkins/jenkins:2.504-jdk17 
USER root
RUN apt-get update && \
    apt-get install -y unzip && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
COPY apache-maven-3.9.9-bin.tar.gz /usr/local/
COPY sonar-scanner-cli-4.8.0.2856-linux.zip /usr/local/
WORKDIR /usr/local/
RUN tar -xzf apache-maven-3.9.9-bin.tar.gz && \
    rm apache-maven-3.9.9-bin.tar.gz
RUN unzip sonar-scanner-cli-4.8.0.2856-linux.zip && \
    rm sonar-scanner-cli-4.8.0.2856-linux.zip
ENV MAVEN_HOME=/usr/local/apache-maven-3.9.9
ENV SONAR_SCANNER_HOME=/usr/local/sonar-scanner-cli-4.8.0.2856
ENV PATH=$JAVA_HOME/bin:$MAVEN_HOME/bin:$SONAR_SCANNER_HOME/bin:$PATH
RUN echo "jenkins ALL=NOPASSWD: ALL" >> /etc/sudoers
USER jenkins
```
### 3.构建镜像 
（[这里以harbor仓库的格式构建后续需要可以推送到harbor](https://github.com/Zorinman/K8S/blob/main/%E9%83%A8%E7%BD%B2%E6%96%87%E6%A1%A3/habor%E9%95%9C%E5%83%8F%E4%BB%93%E5%BA%93.md)）
`docker build -t 192.168.219.129:80/library/jenkins-maven:v1 .`

### 4.构建完成即可查看新生成的镜像
`docker images`

### 5. 推送到harbor   
`docker login 192.168.219.129:80 `  
`docker push 192.168.219.129:80/library/jenkins-maven:v1`  

## 二、利用manifests中的yaml文件部署Jenkins

应用当前目录下的全部yaml资源
`kubectl apply -f .`

**查看是否运行成功**
`kubectl get po -n kube-devops`

**查看 service 端口，通过浏览器访问**
`kubectl get svc -n kube-devops`

 **查看容器日志，查看容器日志，搜索关键词password获取默认登录密码**
`kubectl logs -f pod名称 -n kube-devops`

登录之后我将 用户名和密码改为了 admin 和123 （可以自行去Security修改）

## 三、WebUI上安装所需插件
可以在初次登录时选择自定安装，然后先把自动✔选的安装上
进入主界面之后为了之后的Devops 连接各个中间件，还需要安装以下插件
```
1.Build Authorization Token Root:构建授权 token
2.Gitlab:gitlab 配置插件
3.SonarQube Scanner:代码质量审查工具
4.Node and Label parameter:节点标签参数配置
5.Kubernetes:jenkins + k8s 环境配置插件
6.Config File Provider：用于加载外部配置文件，如 Maven 的 settings.xml 或者 k8s 的 kubeconfig 等
7.Git Paramete：git 参数插件，在进行项目参数化构建时使用

```
