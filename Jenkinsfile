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
def REPO = params.imageRepo
def FAST_PATH = ''

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
                    sh 'wget https://docker-fastpath.s3-eu-west-1.amazonaws.com/releases/linux/docker-fastpath-linux-amd64-latest.tgz'
                    sh 'tar xzvf docker-fastpath-linux-amd64-latest.tgz'
                    sh 'rm docker-fastpath-linux-amd64-latest.tgz'
                    for (String service : services) {
                        withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'docker-hub',
                            usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]) {
                            FAST_PATH = sh(script: "./fastpath --verbose HEAD $REPO", returnStdout: true).trim()
                            if (FAST_PATH == '') {
                                echo "New code. Building..."
                                def imageName = "rs-${service}:latest"
                                echo "Building ${imageName}"
                                def serviceImg = docker.build(imageName)
                                serviceImg.push()
                            } else {
                                echo "No need to rebuild microservice"
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