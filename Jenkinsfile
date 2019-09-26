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

@NonCPS // has to be NonCPS or the build breaks on the call to .each
def build(services) {
    services.each { service ->
        sh '''
            docker build --no-cache -t ${service} .
            docker tag ${service}:latest ${DOCKERHUB_REPO}/${service}:latest
            docker push ${DOCKERHUB_REPO}/${service}:latest
            docker rmi ${service}:latest
        '''   
    }
}
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
        stage('Build') {
            when { expression { env.BRANCH_NAME ==~ /feat.*/ } }
            steps {
                sh 'docker login -u ${DOCKERHUB_USR} -p ${DOCKERHUB_PSW}'
                build(services)
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