pipeline {
    agent any
    environment{
        dockerRegistryUrl = '47.110.58.173:8080/demon'
        imageEndpoint = 'nginx'
        imageTag = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
        image = "${dockerRegistryUrl}/${imageEndpoint}:${imageTag}"
    }

    parameters {
        gitParameter name: 'BRANCH_TAG', 
                     type: 'PT_BRANCH_TAG',
                     branchFilter: 'origin/(.*)',
                     defaultValue: 'master',
                     selectedValue: 'DEFAULT',
                     sortMode: 'DESCENDING_SMART',
		     description: 'Select your branch or tag.'
        choice(name: 'SonarQube', choices: ['False','True'],description: '')				 
        string(name:'BUILD_URL',defaultValue:"http://47.110.58.173:8080",description:'build uri for body')     
    }

  
    stages {

        stage('Checkout SCM') {
            steps {
                checkout([$class: 'GitSCM',
                          branches: [[name: '${params.BRANCH_TAG}']],
                          doGenerateSubmoduleConfigurations: false,
                          extensions: [],
                          submoduleCfg: [],
                          userRemoteConfigs: [[credentialsId: '1c8c17e4-1732-4789-b935-9e8e036cf707', url: 'https://github.com.cnpmjs.org/braior/dockerfile.git']]])
            }
        }

        stage('Build') {
            steps {
                echo 'Start compiling and build'
                sh 'docker build -t ${image} -f Dockerfile .'
            }
        }

        stage('Push') {
            steps {
                withDockerRegistry(credentialsId: 'd5be4e40-5ed7-42f6-a0ae-83c69c71d9ab', url: 'http://47.110.58.173:8080') {
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
            sh './notice -T "text" -m "监控告警：分支: ${BRANCH_TAG} 仓库:  ${BUILD_URL} 版本: ${image}", -t "test生产环境镜像推送成功通知"'
        }
        failure{
            sh './notice -T "text" -m "监控告警：分支: ${BRANCH_TAG} 仓库:  ${BUILD_URL} 版本: ${image}", -t "test生产环境镜像推送失败通知"'
        }
    }
}
