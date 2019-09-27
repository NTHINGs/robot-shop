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
        dir(service) {
            def imageName = "rs-${service}:latest"
            echo "Building ${imageName}"
            def serviceImg = docker.build(imageName)
            serviceImg.push()
        }
    }
}
pipeline {
    agent any
    environment {
        TOKEN = credentials('gh-token')
    }
    triggers {
         pollSCM('* * * * *')
    }
    stages {
        stage('Build') {
            when { expression { env.BRANCH_NAME ==~ /feat.*/ } }
            steps {
                build(services)
            }   
        }   

        stage('deploy') {
          when { branch 'master' }
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