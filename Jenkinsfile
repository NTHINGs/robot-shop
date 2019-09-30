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

podTemplate(label: 'robot-shop', containers: [
    containerTemplate(name: 'docker', image: 'docker', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.8.0', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:latest', command: 'cat', ttyEnabled: true)
  ],
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
  ]) {
    node('master') {

        stage('Build Microservices (if needed)') {
            when { expression { env.BRANCH_NAME ==~ /feat.*/ } }
            container('docker') {

                withCredentials([[$class: 'UsernamePasswordMultiBinding', 
                    credentialsId: 'docker-hub',
                    usernameVariable: 'DOCKER_HUB_USER', 
                    passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
                        for (String service : services) {
                            MICROSERVICE_CHANGED = sh (
                                script: "git diff --name-only $env.GIT_PREVIOUS_COMMIT $env.GIT_COMMIT $service",
                                returnStdout: true
                            ).trim().length() > 0
                            if(MICROSERVICE_CHANGED) {
                                echo "MICROSERVICE $service SOURCE CODE CHANGED. REBUILDING IMAGE"
                                docker.withRegistry( '', 'docker-hub' ) {
                                    dir(service) {
                                        imageName = "nthingsm/rs-$service:latest"
                                        serviceImg = docker.build(imageName)
                                        serviceImg.push()
                                        sh "docker rmi $imageName"
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
                    def repoName = "robot-shop"
                    input(message: "Do you want to create a PR to apply this deployment?", ok: "yes")
                    httpRequest authentication: 'git_user', contentType: 'APPLICATION_JSON_UTF8', httpMode: 'POST', requestBody: """{ "title": "PR Created Automatically by Jenkins", "body": "From Jenkins job: ${env.BUILD_URL}", "head": "nthings:${env.BRANCH_NAME}", "base": "master"}""", url: "https://api.github.com/repos/nthings/${repoName}/pulls"
                }
            }
        }

        stage('do some kubectl work') {
            container('kubectl') {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', 
                        credentialsId: 'docker-hub',
                        usernameVariable: 'DOCKER_HUB_USER',
                        passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
                    
                    sh "kubectl get nodes"
                }
            }
        }
        stage('do some helm work') {
            container('helm') {

               sh "helm ls"
            }
        }
    }
}