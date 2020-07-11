pipeline {
    agent any
    environment{
        DOCKER_HUB_URL = 47.110.58.173:8080
        DOCKER_IMAGE_NAME = nginx_test
        DOCKER_IMAGE_TAG = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
    }
    parameters {
        string(name:'BRANCH_FOR_BODY',defaultValue:"${BRANCH_NAME}",description:'parameters used by ding talk')
        string(name:'BUILD_URL_FOR_BODY',defaultValue:"${BUILD_URL}",description:'build uri for body')
    }  
    stages {
        stage('Checkout') {
            steps {
                echo 'get source code from github or gitlab project'
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '1c8c17e4-1732-4789-b935-9e8e036cf707', url: 'https://github.com.cnpmjs.org/braior/dockerfile.git']]])
            }
        }

        stage('Build') {
            steps {
                echo 'Start compiling and build'
                docker build -t ${DOCKER_HUB_URL}/${DOCKER_IMAGE_NAME}:{DOCKER_IMAGE_TAG} -f Dockerfile .
            }
        }

        stage('Push') {
            steps {
                withDockerRegistry(credentialsId: 'd5be4e40-5ed7-42f6-a0ae-83c69c71d9ab', url: '47.110.58.173:8080') {
                   echo '将本地Docker镜像推送到Harbor镜像仓库' 
                   docker push ${DOCKER_HUB_URL}/${DOCKER_IMAGE_NAME}:{DOCKER_IMAGE_TAG}
                }
            }  
        }

        stage('Clean') {
           steps {
               echo '清理Maven工程'
               sh 'cd hellojib && mvn clean'
               echo '删除镜像'
               sh 'docker rmi bolingcavalry/hellojib:0.0.1-SNAPSHOT 192.168.50.167/library/hellojib:0.0.1-SNAPSHOT'
               echo '清理完毕'
           }
        } 
    post {
        success {
          sh 'sh notice.sh "test生产环境镜像推送成功通知" "nginx-test" "推送镜像成功"  ${BRANCH_NAME} ${BUILD_URL}'
        }
        failure{
          sh 'sh notice.sh "test生产环境镜像推送失败通知" "nginx-test" "推送镜像失败"  ${BRANCH_NAME} ${BUILD_URL}'
        }
    }
}
