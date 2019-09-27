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
    }
    triggers {
         pollSCM('H/5 * * * *')
    }
    stages {
        stage('Build') {
            when { expression { env.BRANCH_NAME ==~ /feat.*/ } }
            steps {
                script {
                    for (String service : services) {
                        MICROSERVICE_CHANGED = sh (
                            script: "git diff --name-only $env.GIT_PREVIOUS_COMMIT $env.GIT_COMMIT $service",
                            returnStatus: true
                        ) == 0
                        echo "MICROSERVICE SOURCE CODE CHANGED: $MICROSERVICE_CHANGED"
                        docker.withRegistry( '', 'docker-hub' ) {
                            dir(service) {
                                def imageName = "nthingsm/rs-$service:latest"
                                def serviceImg = docker.build(imageName)
                                serviceImg.push()
                                sh "docker rmi $imageName"
                            }
                        }
                    }
                }
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