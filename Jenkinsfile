pipeline {
    agent any

    environment {
        NODE_OPTIONS = "--dns-result-order=ipv4first"
    }
    stages {
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
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
    }
}