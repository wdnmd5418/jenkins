// 镜像仓库地址
def registry = "registry.cn-qingdao.aliyuncs.com"
// 命名空间
def namespace = "lantianbaiyun"
// 镜像仓库项目
def project = "gitlab-pipeline"
// 镜像名称
def app_name = "citest"
// 镜像完整名称
def image_name = "${registry}/${namespace}/${project}:${app_name}-${BUILD_NUMBER}"
// git仓库地址
def git_address = "https://github.com/wdnmd5418/jenkins.git"
// fenzhi分支
def branch = "*/master"

// 认证
def aliyunhub_auth = "8871e929-daa9-45ee-b339-e3d6df94e15f"
def gitlab_auth = "cbba63e0-65ef-4675-8b38-aad30a9d2d64"


// K8s认证
def k8s_auth = "266123aa-1108-4df2-88be-3760a6c23df2"
// aliyun仓库secret_name
def aliyun_registry_secret = "aliyun-pull-secret"
// k8s部署后暴露的nodePort
def nodePort = "30666"


podTemplate(
    label: 'jenkins-agent', 
    cloud: 'kubernetes',
    namespace: "kube-ops",
    containers: [
       
    ]){
    node('jenkins-agent'){
        stage('拉取代码') { // for display purposes
            checkout([$class: 'GitSCM',branches: [[name: '*/master']], userRemoteConfigs: [[credentialsId: "${gitlab_auth}", url: "${git_address}"]]])
        }
        stage('代码编译') {
        //    sh "mvn clean package -Dmaven.test.skip=true"
            sh "ls"
        }
        stage('构建镜像') {
            container('docker') {
                stage('打包镜像') {
                   withCredentials([usernamePassword(credentialsId: "${aliyunhub_auth}", passwordVariable: 'password', usernameVariable: 'username')]) {
                   sh """
                      docker build -t ${image_name} .
                      docker login -u ${username} -p '${password}' ${registry}
                      docker push ${image_name}
                   """
                    }
                }  
            }    
        }
        stage('部署到K8s'){
            sh """
                sed -i 's#\$IMAGE_NAME#${image_name}#' deployment.yaml
                sed -i 's#\$SECRET_NAME#${aliyun_registry_secret}#' deployment.yaml
                sed -i 's#\$NODE_PORT#${nodePort}#' deployment.yaml
            """
            kubernetesDeploy configs: 'deployment.yaml', kubeconfigId: "${k8s_auth}"
        }
    }
}
