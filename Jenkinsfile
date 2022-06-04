pipeline {
    agent any 
    options {
      skipStagesAfterUnstable()
    }
    stages {
        stage('Clone repository') {
          steps {
            script { 
              checkout scm
            }
          }
        }
        stage('Build') {
          steps {
            script {
              app = docker.build("hello-nginx")
            }
          }
        }
        stage('Test') {
          steps {
            echo 'Empty'
          }
        }
        stage('Push') {
          steps {
            script {
              docker.withRegistry(
                'https://281292694180.dkr.ecr.us-east-1.amazonaws.com',
                'ecr:us-east-1:my.aws.credentials') {
                  app.push("${env.BUILD_NUMBER}")
                  app.push("latest")
                }
            }
          }
        }
        stage('Deploy') {
            steps {
                echo 'Deploy to Fargate' 
            }
        }
    }
}
