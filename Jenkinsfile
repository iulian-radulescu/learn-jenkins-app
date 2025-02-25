pipeline {
    agent {
        docker {
            image 'node:18.18.2-alpine'
                    args '--network=host'
                    reuseNode true
        }
    }

    stages {
        stage('Build') {

            steps {
                sh '''
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

        stage ('Test') {
            sh '''
                cat build/index.html
                npm test
            '''
        }
    }
}