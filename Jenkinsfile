pipeline {
   agent any

   environment {
        REACT_APP_VERSION="1.2.${BUILD_ID}"
        AWS_DEFAULT_REGION="eu-central-1"
    }

   stages {
        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "--entrypoint='' --network=host"
                    reuseNode true
                }
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws-access-credentials', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    // some block
                     sh '''
                        aws --version
                        aws ecs register-task-definition --cli-input-json file://aws/task-definition.json
                     '''
                }
            }
        }

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
    }
}