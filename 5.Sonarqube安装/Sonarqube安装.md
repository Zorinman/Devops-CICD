| 步骤 | 描述                                   |
|------|----------------------------------------|
| 1    | [动态构建 PV 和 PVC](1.创建动态构建PV，PVC/动态构建Pv,pvc.md)                     |
| 2    | [安装 GitLab](2.gitlab安装/gitlab安装.md)                            |
| 3    | [将 Git 关联到 GitLab 并推送项目](3.使用git推送项目到gitlab/使用git推送项目到gitlab.md)        |
| 4    | [安装 Harbor](4.Harbor安装/Harbor安装.md)                           |
| 5    |➡️ [安装 SonarQube](5.Sonarqube安装/Sonarqube安装.md)                         |
| 6    | [安装 Jenkins](6.jenlinks安装/jenlinks安装.md)                           |
| 7    | [连接各个中间件形成 DevOps](7.连接各个中间件形成devops/连接中间件形成Devops.md)         |
| 8    | [创建流水线任务](8.创建pipe流水线/创建流水线任务.md)    

**这里把pgsql.yaml文件和sonarqube.yaml放在master节点的/opt/k8s/devops/sonarqube进行安装**

⭐创建一个名为`kube-devops`的命名空间，之后Sonarqube,jenkins,pgsql都会部署在这个命名空间里

部署yaml资源安装 `kubeclt apply -f .`


注意这里使用持久化存储，匹配使用动态构建pv,pvc的模式的storageclass  需要提前配置好Provisioner StorageClass RBAC  ,我们已经配置好

**查看对应命名空间下的Sonarqube服务**
复制映射的外部端口  因为是Nodeport形式，所以使用任意集群IP+端口号就可访问

账号密码在 默认都为admin
这里我把密码改为!Zhoulinguang987


## 安装中文插件


![alt text](图片/image.png)