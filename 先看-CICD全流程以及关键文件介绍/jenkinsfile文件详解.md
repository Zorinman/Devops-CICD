几个坑：
1.因为已经在jenkins的容器内部了，所以不用在额外指定什么container("maven")之类的  
2.gitlab上项目之后部署到K8S集群中的那个命名空间必须提前创建，可以自己创建可以在该Jenkinsfile中创建。


```groovy
pipeline {
  agent  any //表示流水线可以在任何可用的 Jenkins 节点上运行。这里默认在jenkins master 容器内
  stages {
    stage('clone code') { //根据凭证 从 GitLab 仓库克隆代码
      steps {
          git(url: 'http://192.168.219.149:22222/gitlab-instance-0cafae89/k8s-cicd-demo.git', credentialsId: 'gitlab-user-pass', branch: '$BRANCH_NAME', changelog: true, poll: false)
      }
    }
	
    

    stage('unit test') {  //使用 Maven 执行单元测试。clean 命令会清理先前的构建，test 会运行测试套件
      steps {
          sh 'mvn clean test'
      }
    }
 
    stage('sonarqube analysis') { //使用凭据 ID sonarqube-token 使用 SonarQube 进行代码分析
      agent none
      steps {
        withCredentials([string(credentialsId : 'sonarqube-token' ,variable : 'SONAR_TOKEN' ,)]) {
          withSonarQubeEnv('sonarqube') {
              sh '''mvn sonar:sonar -Dsonar.projectKey=$APP_NAME 
echo "mvn sonar:sonar -Dsonar.projectKey=$APP_NAME"'''// 运行 Maven 的 SonarQube 插件，进行代码质量分析

          }

          timeout(unit: 'MINUTES', activity: true, time: 5) {
            waitForQualityGate 'true'
          }

        }

      }
    }
  

    stage('build & push') {
      steps {
          sh 'mvn clean package -DskipTests'
          sh 'docker build -f Dockerfile -t $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BUILD_NUMBER .'
          withCredentials([usernamePassword(credentialsId : 'harbor-user-pass' ,passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,)]) {
            sh '''echo "$DOCKER_PASSWORD" | docker login $REGISTRY -u "$DOCKER_USERNAME" --password-stdin
docker push $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BUILD_NUMBER'''
          }


      }
    }

    stage('push latest') {
      when {
        branch 'main'//只有在 main 分支时才会执行这一阶段
      }
      steps {
          sh 'docker tag $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BUILD_NUMBER $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:latest'//将 SNAPSHOT 标签的镜像重命名为 latest 标签
          sh 'docker push $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:latest'

      }
    }

    stage('deploy to dev') {
      steps {
          input(id: 'deploy-to-dev', message: 'deploy to dev?')//等待用户确认是否继续部署到开发环境
          configFileProvider([configFile(fileId: '334d10b5-4bb1-46fb-80e2-8f0c132bf3e8', targetLocation: '/tmp/admin.kubeconfig')]) {//从jenkins上加载 Kubernetes 配置文件 以admin.kubeconfig保存到临时目录，之后将其放到容器内的 ~/.kube/config 中
            sh 'mkdir -p ~/.kube/'
            sh 'cp /tmp/admin.kubeconfig ~/.kube/config'
			sh 'kubectl create namespace ks-k8s-cicd-demo || echo "Namespace already exists"'  //提前创建cicd-demo-dev.yaml里面定义的namespace，不然会部署不了
            sh '''sed -i\'\' "s#REGISTRY#$REGISTRY#" deploy/cicd-demo-dev.yaml
sed -i\'\' "s#DOCKERHUB_NAMESPACE#$DOCKERHUB_NAMESPACE#" deploy/cicd-demo-dev.yaml 
sed -i\'\' "s#APP_NAME#$APP_NAME#" deploy/cicd-demo-dev.yaml
sed -i\'\' "s#BUILD_NUMBER#$BUILD_NUMBER#" deploy/cicd-demo-dev.yaml
kubectl apply -f deploy/cicd-demo-dev.yaml''' //动态替换 deploy/cicd-demo-dev.yaml 文件中的占位符（如 REGISTRY、APP_NAME、BUILD_NUMBER 等）为环境变量的实际值
          }


      }
    }

    stage('push with tag') { //仅在 TAG_NAME 以 v 开头时才执行该阶段。
      agent none
      when {
        expression {
          params.TAG_NAME =~ /v.*/
        }

      }
      steps {
        input(message: 'release image with tag?', submitter: '')
        withCredentials([usernamePassword(credentialsId : 'gitlab-user-pass' ,passwordVariable : 'GIT_PASSWORD' ,usernameVariable : 'GIT_USERNAME' ,)]) { //使用凭据 ID harbor-user-pass 获取 harbor 注册表的用户名和密码，并登录到 harbor 仓库
          sh 'git config --global user.email "910254930@qq.com" '
          sh 'git config --global user.name "Zorin" '
          sh 'git tag -a $TAG_NAME -m "$TAG_NAME" '
          sh 'git push http://$GIT_USERNAME:$GIT_PASSWORD@$GIT_REPO_URL/$GIT_ACCOUNT/k8s-cicd-demo.git --tags --ipv4'
        }

          sh 'docker tag $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BUILD_NUMBER $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:$TAG_NAME'
          sh 'docker push $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:$TAG_NAME'

      }
    }

    stage('deploy to production') {//仅在 TAG_NAME 以 v 开头时执行该阶段。
      agent none
      when {
        expression {
          params.TAG_NAME =~ /v.*/
        }

      }
      steps {
        input(message: 'deploy to production?', submitter: '')
          sh '''sed -i\'\' "s#REGISTRY#$REGISTRY#" deploy/cicd-demo.yaml
sed -i\'\' "s#DOCKERHUB_NAMESPACE#$DOCKERHUB_NAMESPACE#" deploy/cicd-demo.yaml
sed -i\'\' "s#APP_NAME#$APP_NAME#" deploy/cicd-demo.yaml
sed -i\'\' "s#TAG_NAME#$TAG_NAME#" deploy/cicd-demo.yaml

kubectl apply -f deploy/cicd-demo.yaml'''

      }
    }

  }
  environment {  //定义了用于整个流水线的环境变量，方便在流水线中多次使用这些值
    REGISTRY = '192.168.219.129:80'  //harbor仓库地址
    DOCKER_CREDENTIAL_ID = 'harbor-user-pass'  //harbor仓库的访问凭证,jenkins上配置了
    GIT_REPO_URL = 'http://192.168.219.149:22222'//gitlab的地址
    GIT_CREDENTIAL_ID = 'gitlab-user-pass'//gitlab的访问凭证，jenkins上配置了
    KUBECONFIG_CREDENTIAL_ID = '334d10b5-4bb1-46fb-80e2-8f0c132bf3e8'//jenkins上加载的kubeconfig文件的ID
    DOCKERHUB_NAMESPACE = 'library'//harbor仓库下你的目录空间
    GITHUB_ACCOUNT = 'root'
    APP_NAME = 'k8s-cicd-demo'//项目的名字，构建镜像等等会用到
  }
  parameters {
    string(name: 'BRANCH_NAME', defaultValue: 'main', description: '请选择要发布的分支')  //自己的分支
    string(name: 'TAG_NAME', defaultValue: 'snapshot', description: '标签名称，必须以 v 开头，例如：v1、v1.0.0')
  }
}


```