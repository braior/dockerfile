pipeline {
    agent any
    environment{
        dockerRegistryUrl = '47.110.58.173:8080'
        imageEndpoint = 'nginx_test'
        imageTag = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
        image = "${dockerRegistryUrl}/${imageEndpoint}:${imageTag}"
    }
    parameters {
        string(name:'BRANCH_FOR_BODY',defaultValue:"${BRANCH_NAME}",description:'parameters used by ding talk')
        string(name:'BUILD_URL_FOR_BODY',defaultValue:"${BUILD_URL}",description:'build uri for body')
    }  
    stages {
        stage('Build') {
            steps {
                echo 'Start compiling and build'
                sh 'docker build -t ${image} -f Dockerfile .'
            }
        }

        stage('Push') {
            steps {
                withDockerRegistry(credentialsId: 'd5be4e40-5ed7-42f6-a0ae-83c69c71d9ab', url: '47.110.58.173:8080') {
                   echo '将本地Docker镜像推送到Harbor镜像仓库' 
                   sh 'docker push ${image}'
                }
            }  
        }

        stage('Clean') {
           steps {
               echo '删除镜像'
               sh 'docker rmi ${image}'
               echo '清理完毕'
           }
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
