def DOCKERHUB_REPO = 'nthingsm'
def services = [
    'mongodb',
    'catalogue',
    'user',
    'cart',
    'mysql',
    'shipping',
    'ratings',
    'payment',
    'dispatch',
    'web'
]
pipeline {
    agent any
    environment {
        TOKEN = credentials('gh-token')
        DOCKERHUB = credentials('dockerhub')
    }
    triggers {
         pollSCM('* * * * *')
    }
    stages {
        for (int i = 0; i < services.size(); i++) {
            stage('Build ${services[i]}'){ 
                when { expression { env.BRANCH_NAME ==~ /feat.*/ } }
                steps {
                    sh '''
                        docker login -u ${DOCKERHUB_USR} -p ${DOCKERHUB_PSW}
                        docker build --no-cache -t ${services[i]} .
                        docker tag ${services[i]}:latest ${DOCKERHUB_REPO}/${services[i]}:latest
                        docker push ${DOCKERHUB_REPO}/${services[i]}:latest
                        docker rmi ${services[i]}:latest
                    '''    
                }
            }
        }

        stage('deploy'){
          when { expression { env.BRANCH_NAME ==~ /master/ } }
          steps {
              sh 'deploy services to k8s'
          }
        }
    }
    post {
      success {
          echo 'success'
      }
      failure {
           echo 'FAILED'
      }
    }
}