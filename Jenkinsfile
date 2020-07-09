pipeline {
  agent any
  parameters {
     string(name:'BRANCH_FOR_BODY',defaultValue:"${BRANCH_NAME}",description:'parameters used by ding talk')
     string(name:'BUILD_URL_FOR_BODY',defaultValue:"${BUILD_URL}",description:'build uri for body')
  }
  stages {
    stage('Deploy-Production') {
      when {
        branch 'master'
      }
      steps {
        sh '''
         docker build -t ${DOCKER_IMAGE_NAME}:latest -f Dockerfile .                                     
        '''
        sh 'docker push ${DOCKER_IMAGE_NAME}:latest'
        sh 'docker rmi ${DOCKER_IMAGE_NAME}:latest'
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
  }
  environment {
    DOCKER_DEPLOY_URI = 'http://nginx.test.shiji.com'
    DOCKER_IMAGE_NAME = 'docker-hub.nooboh.com/alpine_nginx'
    DOCKER_SERVICE_NAME = 'TestService'
  }
}
