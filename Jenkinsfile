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

def image_tag = "latest"

def buildImage(String repo, String service, String tag) {
    imageName = "$repo/rs-$service:$tag"
    serviceImg = docker.build(imageName, '--network=host .')
    serviceImg.push()
    sh "docker rmi $imageName"
}

pipeline {
    agent any
    triggers {
         pollSCM('H/5 * * * *')
    }
    environment {
        K8S_URL = credentials('k8s-url')
    }
    stages {
        stage('Build') {
            when { expression { env.BRANCH_NAME ==~ /feat.*/ } }
            steps {
                script {
                    master_latest = sh (
                        script: "git rev-parse origin/master",
                        returnStdout: true
                    ).trim()
                    for (String service : services) {
                        MICROSERVICE_CHANGED = sh (
                            script: "git diff --name-only $master_latest $env.GIT_COMMIT $service",
                            returnStdout: true
                        ).trim().length() > 0
                        if(MICROSERVICE_CHANGED) {
                            echo "MICROSERVICE $service SOURCE CODE CHANGED. REBUILDING IMAGE"
                            docker.withRegistry( '', 'docker-hub' ) {
                                dir(service) {
                                    image_tag = sh (
                                        script: "git rev-parse --short HEAD",
                                        returnStdout: true
                                    ).trim()

                                    buildImage("nthingsm", service, image_tag)
                                }
                            }
                        } else {
                            echo "No need to rebuild this microservice $service."
                        }
                    }
                }
            }   
        }   
        stage('pull-request') {
            when { expression { env.BRANCH_NAME ==~ /feat.*/ } }
            steps {
                script {
                    timeout(time: 15, unit: 'MINUTES') { 
                        def repoName = "robot-shop"
                        input(message: "Do you want to create a PR to apply this deployment?", ok: "yes")
                        httpRequest authentication: 'git_user', contentType: 'APPLICATION_JSON_UTF8', httpMode: 'POST', requestBody: """{ "title": "PR Created Automatically by Jenkins", "body": "From Jenkins job: ${env.BUILD_URL}", "head": "nthings:${env.BRANCH_NAME}", "base": "master"}""", url: "https://api.github.com/repos/nthings/${repoName}/pulls"
                    }
                }
            }
        }

        stage('deploy') {
            when { branch 'master' }
            steps {
                script {
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', 
                            credentialsId: 'docker-hub',
                            usernameVariable: 'DOCKER_HUB_USER',
                            passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
                        withKubeConfig([credentialsId: 'kubeconfig',
                            serverUrl: "$K8S_URL",
                            namespace: 'mauricio']) {
                                //sh "kubectl apply -f K8s/descriptors -n mauricio"
                                sh "helm init --tiller-namespace mauricio "
                                image_tag = sh (
                                    script: "git rev-parse --short HEAD",
                                    returnStdout: true
                                ).trim()
                                sh "helm upgrade --namespace=mauricio --tiller-namespace mauricio --install robot-shop helm-robot-shop --set ImageTag=$image_tags"
                            }
                    }
                }   
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