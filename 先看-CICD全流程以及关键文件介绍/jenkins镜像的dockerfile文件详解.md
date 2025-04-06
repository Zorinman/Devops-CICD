在Jenkins镜像集成了maven和sonarqube，为之后直接在这个部署在Jenkins master容器中执行流水线任务打下基础
```dockerfile
FROM jenkins/jenkins:2.504-jdk17 
# 使用官方的 Jenkins 镜像，版本为 2.504，基于 JDK 17。

USER root
# 切换到 root 用户，以便执行需要管理员权限的操作。

RUN apt-get update && \
    apt-get install -y unzip && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
# 更新 apt 包索引，安装 unzip 工具，用于解压文件。
# 清理 apt 缓存以减少镜像体积。

COPY apache-maven-3.9.9-bin.tar.gz /usr/local/
# 将 Maven 的压缩包复制到容器的 /usr/local/ 目录。

COPY sonar-scanner-cli-4.8.0.2856-linux.zip /usr/local/
# 将 SonarQube Scanner 的压缩包复制到容器的 /usr/local/ 目录。

WORKDIR /usr/local/
# 设置工作目录为 /usr/local/，后续命令将在该目录下执行。

RUN tar -xzf apache-maven-3.9.9-bin.tar.gz && \
    rm apache-maven-3.9.9-bin.tar.gz
# 解压 Maven 压缩包，并删除原始压缩文件以节省空间。

RUN unzip sonar-scanner-cli-4.8.0.2856-linux.zip && \
    rm sonar-scanner-cli-4.8.0.2856-linux.zip
# 解压 SonarQube Scanner 压缩包，并删除原始压缩文件以节省空间。

ENV MAVEN_HOME=/usr/local/apache-maven-3.9.9
# 设置 Maven 的安装路径为环境变量 MAVEN_HOME。

ENV SONAR_SCANNER_HOME=/usr/local/sonar-scanner-cli-4.8.0.2856
# 设置 SonarQube Scanner 的安装路径为环境变量 SONAR_SCANNER_HOME。

ENV PATH=$JAVA_HOME/bin:$MAVEN_HOME/bin:$SONAR_SCANNER_HOME/bin:$PATH
# 将 Java、Maven 和 SonarQube Scanner 的可执行文件路径添加到 PATH 环境变量中。

RUN echo "jenkins ALL=NOPASSWD: ALL" >> /etc/sudoers
# 配置 sudo 权限，允许 Jenkins 用户在不输入密码的情况下执行 sudo 命令。

USER jenkins
# 切换回 Jenkins 用户，提升安全性，避免使用 root 用户执行不必要的操作。
```
```dockerfile
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
### Dockerfile 详解

#### 基础镜像
- **FROM**: 使用 `jenkins/jenkins:2.504-jdk17` 作为基础镜像。

#### 文件复制
- **COPY**: 将以下文件复制到容器的 `/usr/local/` 目录：
    - `apache-maven-3.9.9-bin.tar.gz`：Maven 3.9.9 的二进制压缩包。
    - `sonar-scanner-cli-4.8.0.2856-linux.zip`：SonarQube Scanner 4.8.0 的压缩包。

#### 工作目录
- **WORKDIR**: 切换到 `/usr/local/` 目录，后续操作将在该目录下执行。

#### 解压操作
- **Maven 解压**: 解压 `apache-maven-3.9.9-bin.tar.gz`，解压后生成 `apache-maven-3.9.9` 目录。
- **SonarQube Scanner 解压**: 解压 `sonar-scanner-cli-4.8.0.2856-linux.zip`，解压后生成 `sonar-scanner-cli-4.8.0.2856` 目录。

#### 环境变量
- **MAVEN_HOME**: 设置为 `/usr/local/apache-maven-3.9.9`。
- **SONAR_SCANNER_HOME**: 设置为 `/usr/local/sonar-scanner-cli-4.8.0.2856`。
- **PATH**: 更新 PATH 环境变量，将 Java、Maven 和 SonarQube Scanner 的可执行文件路径加入到系统的环境变量中，这样可以在命令行直接执行它们的命令。

#### 用户权限
- **USER root**: 切换到 root 用户，执行需要管理员权限的操作（如安装软件包）。
- **USER jenkins**: 切换回 Jenkins 用户，提升安全性，避免使用 root 用户执行不必要的操作。
