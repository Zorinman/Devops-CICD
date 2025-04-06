(当前)动态构建PV，PVC创建→gitlab安装→git关联到gitlab并推送项目→harbor安装→sonarqube安装→jenkins安装→连接各个中间件形成Devops→创建流水线任务

之后Jenkins,Sonarqube,postgres会使用该制备器来构建持久化存储 

参考[PV,PVC的动态与静态构建](https://github.com/Zorinman/K8S/blob/main/%E9%83%A8%E7%BD%B2%E6%96%87%E6%A1%A3/PV%2CPVC%E7%9A%84%E5%8A%A8%E6%80%81%E4%B8%8E%E9%9D%99%E6%80%81%E6%9E%84%E5%BB%BA.md)
这里使用NFS的卷存储类型 
## 1.[NFS安装与配置](https://github.com/Zorinman/K8S/blob/main/%E9%83%A8%E7%BD%B2%E6%96%87%E6%A1%A3/NFS%E5%AE%89%E8%A3%85%E4%B8%8E%E9%85%8D%E7%BD%AE.md)

## 2.创建Provisioner、StroageClass、RBAC配置
（这里PVC在之后Jenkins,Sonarqube,postgres的pvc.yaml里创建,pvc创建之后pv会自动创建）

应用三个yaml文件 
`kubectl apply  -f .`

