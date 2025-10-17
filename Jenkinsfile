pipeline {
    agent any

    environment {
        BUILD_DIR = 'build'
        INDEX_HTML = 'index.html'
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        AWS_DEFAULT_REGION = "us-east-1"
        AWS_ECS_CLUSTER = 'LearnJenkinsApp-Production-Cluster'
        AWS_ECS_SERVICE = 'LearnJenkinsApp-TaskDefinition-Prod-Service'
        AWS_ECS_TASKDEF = 'LearnJenkinsApp-TaskDefinition-Prod'
    }
 
    stages {

        stage('Build') {
            agent {
                docker {
                    image 'my-docker'
                    reuseNode true
                }
            }
 
            steps {
                sh '''
                    ls -la
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
 
        stage ('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                            aws --version
                            yum install jq -y
                            LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                            echo $LATEST_TD_REVISION
                            aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE --task-definition $AWS_ECS_TASKDEF:$LATEST_TD_REVISION
//                            aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE
                        '''
                }
            }
        }

        stage('Build MyJenkinsApp Docker Image') {
            steps {
                sh 'docker build -t my-jenkins-app  .'
            }
        }
     }
}