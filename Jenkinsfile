pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        APP_NAME = 'learnjenkinsapp'
        AWS_DEFAULT_REGION = 'us-east-1'
        AWS_ECS_CLUSTER = 'learnjenkinsapp-cluster-prod'
        AWS_ECS_SERVICE_PROD = 'LearnJenkinsApp-TaskDefinition-Prod-service-q88y8qgu'
        AWS_ECS_TD_PROD = 'LearnJenkinsApp-TaskDefinition-Prod'
        AWS_DOCKER_REGISTORY = '591950085577.dkr.ecr.us-east-1.amazonaws.com'
    }

    stages {



        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Build docker image') {
            agent {
                docker{
                    image 'my-aws-cli'
                    reuseNode true
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {

                    sh '''
                        yum install -y docker
                        docker build -t $AWS_DOCKER_REGISTORY/$APP_NAME:$REACT_APP_VERSION .
                        aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTORY
                        docker push $AWS_DOCKER_REGISTORY/$APP_NAME:$REACT_APP_VERSION
                    '''
                }
            }
        }

        stage('Deploy to aws'){
            agent {
                docker{
                    image 'my-aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }

            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        sed -i "s/#APP_VERSION#/$REACT_APP_VERSION/g" aws/task-definition-prod.json
                        yum install jq -y
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                        aws ecs update-service  --cluster $AWS_ECS_CLUSTER  --service $AWS_ECS_SERVICE_PROD --task-definition $AWS_ECS_TD_PROD:$LATEST_TD_REVISION
                        aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE_PROD
                    '''
                }
            }
        }
    }
}
