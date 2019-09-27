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
                        echo env.GIT_PREVIOUS_COMMIT
                        echo env.GIT_COMMIT
                        MICROSERVICE_CHANGED = sh (
                            script: "git diff --name-only $env.GIT_PREVIOUS_COMMIT $env.GIT_COMMIT $service",
                            returnStdout: true
                        ).trim().length() > 0
                        if(MICROSERVICE_CHANGED) {
                            echo "MICROSERVICE SOURCE CODE CHANGED: $MICROSERVICE_CHANGED"
                            docker.withRegistry( '', 'docker-hub' ) {
                                dir(service) {
                                    def imageName = "nthingsm/rs-$service:latest"
                                    def serviceImg = docker.build(imageName)
                                    serviceImg.push()
                                    sh "docker rmi $imageName"
                                }
                            }
                        } else {
                            echo "No need to rebuild this microservice."
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