pipeline {
    agent any 
    options {
      skipStagesAfterUnstable()
    }
    environment {
      REPOSITORY_URI = "281292694180.dkr.ecr.us-east-1.amazonaws.com"
      SERVICE_NAME = "hello-nginx"
      TASK_FAMILY = "hello-nginx"
      DESIRED_COUNT ="1"
      CLUSTER_NAME = "hello-nginx"
      AWS_ID = credentials("my.aws.credentials")
      AWS_ACCESS_KEY_ID = "${env.AWS_ID_USR}"
      AWS_SECRET_ACCESS_KEY = "${env.AWS_ID_PSW}"
      AWS_DEFAULT_REGION = "us-east-1"
      EXECUTION_ROLE_ARN = "arn:aws:iam::281292694180:role/ecsTaskExecutionRole"
    }
    stages {
        stage('Clone repository') {
          steps {
            script { 
              checkout scm
            }
          }
        }
        stage('Build Image') {
          steps {
            script {
              app = docker.build("hello-nginx")
            }
          }
        }
        stage('Test Image') {
          steps {
            echo 'Empty'
          }
        }
        stage('Push Image') {
          steps {
            script {
              docker.withRegistry(
                "https://$REPOSITORY_URI",
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

                // prepare task definition file
                sh """sed -e "s;%REPOSITORY_URI%;${REPOSITORY_URI};g" -e "s;%TAG%;${env.BUILD_NUMBER};g" -e "s;%TASK_FAMILY%;${TASK_FAMILY};g" -e "s;%SERVICE_NAME%;${SERVICE_NAME};g" -e "s;%EXECUTION_ROLE_ARN%;${EXECUTION_ROLE_ARN};g" taskdef.json > taskdef-${env.BUILD_NUMBER}.json"""

                script {
                  withAWS(region: 'us-east-1', credentials: 'my.aws.credentials') {
                    taskDefRegistry = readJSON text: sh(returnStdout: true, script:"aws ecs register-task-definition --output json --cli-input-json file://taskdef-${env.BUILD_NUMBER}.json > ${env.WORKSPACE}/temp.json"), returnPojo: true
                    def projects = readJSON file: "${env.WORKSPACE}/temp.json"
                    def TASK_REVISION = projects.taskDefinition.revision

                    def updateService = "aws ecs update-service --service ${SERVICE_NAME} --cluster ${CLUSTER_NAME} --task-definition ${TASK_FAMILY}:${TASK_REVISION} --desired-count ${DESIRED_COUNT} --force-new-deployment"
                    def runUpdateService = sh(returnStdout: true, script: updateService)
                    def serviceStable = "aws ecs wait services-stable --service ${SERVICE_NAME} --cluster ${CLUSTER_NAME}"
                    sh(returnStdout: true, script: serviceStable)
                    // put all your slack messaging here
                  }
                }

            }
        }
    }
}
