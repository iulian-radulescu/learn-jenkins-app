pipeline {
   agent any

   environment {
        REACT_APP_VERSION="1.2.${BUILD_ID}"
        AWS_DEFAULT_REGION="eu-central-1"
        AWS_ECS_CLUSTER="LearnJenkinsApp-Cluster-Prod"
        AWS_ECS_SERVICE="LearnJenkinsApp-Service-Prod"
        AWS_ECS_TD="LearnJenkinsApp-TaskDefinition-Prod"
    }

   stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18.18.2-alpine'
                    args '--network=host'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Change"
                    ls -la
                    node --version
                    npm --version
                    npm cache clean --force
                    npm install --verbose
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Build Docker Image') {
            agent {
                docker {
                    image 'my-aws-cli'
                    args "--entrypoint='' --network=host -u root -v /var/run/docker.sock:/var/run/docker.sock"
                    reuseNode true
                }
            }
            steps {
                sh '''
                   docker build -t learn-jenkins-app --network=host .
                '''
                sh ''
            }
        }

         stage('Deploy to AWS') {
            agent {
                docker {
                    image 'my-aws-cli'
                    args "--entrypoint='' --network=host -u root"
                    reuseNode true
                }
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws-access-credentials', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    // some block
                     sh '''
                        aws --version
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition.json | jq '.taskDefinition.revision')
                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE--task-definition $AWS_ECS_TD:$LATEST_TD_REVISION
                        aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE
                     '''
                }
            }
        }
    }
}