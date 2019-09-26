pipeline {
    def REPO = 'nthingsm'
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
    agent any
    environment {
        TOKEN = credentials('gh-token')
        DOCKERHUB = credentials('dockerhub')
    }
    triggers {
         pollSCM('* * * * *')
    }
    stages {
        services.each { service ->
            stage('Build ${service}'){ 
                when { expression { env.BRANCH_NAME ==~ /feat.*/ } }
                steps {
                    sh '''
                        docker login -u ${DOCKERHUB_USR} -p ${DOCKERHUB_PSW}
                        docker build --no-cache -t ${service} .
                        docker tag ${service}:latest ${REPO}/${service}:latest
                        docker push ${REPO}/${service}:latest
                        docker rmi ${service}:latest
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