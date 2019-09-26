pipeline {
    agent any
    environment {
        TOKEN = credentials('gh-token')
    }
    triggers {
         pollSCM('H/5 * * * *')
    }
    stages {
        stage('init'){
          when { expression { env.BRANCH_NAME ==~ /feat.*/ } }
          steps{
            sh 'echo "IT WORKS"'    
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