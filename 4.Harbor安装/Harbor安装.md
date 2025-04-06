动态构建PV，PVC创建→gitlab安装→(当前)→git关联到gitlab并推送项目→harbor安装→sonarqube安装→jenkins安装→连接各个中间件形成Devops→创建流水线任务

[Harbor安装](https://github.com/Zorinman/K8S/blob/main/%E9%83%A8%E7%BD%B2%E6%96%87%E6%A1%A3/habor%E9%95%9C%E5%83%8F%E4%BB%93%E5%BA%93.md)

注意，之后会使用pod从harbor仓库拉取和推送镜像 ，为避免繁琐证书配置，这里配置容器运行时使用Http协议而不是Https协议访问harbor

## 如果K8s的容器运行时是docker  
则只需要在`/etc/daemon.json`中配置`"insecure-registries": ["192.168.219.129:80"]` 就可以使用Http协议访问   

## 如果K8s的容器运行时containerd则配置（我的K8S容器运行时）：  
`vim /etc/containerd/config.toml`  
```yaml
[plugins."io.containerd.grpc.v1.cri".registry]
  config_path = ""  # 保留空字符串（或删除此行）

  [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."192.168.219.129:80"] #新增,仓库地址
      endpoint = ["http://192.168.219.129:80"]  # 新增,显式指定 HTTP 协议

  [plugins."io.containerd.grpc.v1.cri".registry.configs]
    [plugins."io.containerd.grpc.v1.cri".registry.configs."192.168.219.129:80".tls] #新增
      insecure_skip_verify = true  # 新增，关键： Harbor 使用 HTTP，必须配置来跳过TLS证书验证

  # 以下为空区块可删除（除非需要特殊配置）
  [plugins."io.containerd.grpc.v1.cri".registry.auths]
  [plugins."io.containerd.grpc.v1.cri".registry.headers]
```
重启containerd `systemctl restart containerd` 
