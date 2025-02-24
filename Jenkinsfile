pipeline {
    agent any

    stages {
        stage('CleanUp') {
            steps {
                cleanWs();
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'node:18.18.2-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm install
                    npm run build
                    ls -la
                '''
            }
        }
    }
}