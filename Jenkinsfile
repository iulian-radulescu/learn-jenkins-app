pipeline {
   agent any

    stages {

        //stage('Clean') {
        //    steps {
        //        cleanWs()
        //    }
        //}

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

        stage('Running Tests') {
            parallel {
                stage ('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18.18.2-alpine'
                            args '--network=host'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            echo "Test Stage"
                            cat build/index.html
                            npm test
                        '''
                    }

                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                 stage ('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            args '--network=host'
                            reuseNode true
                        }
                    }

                    steps {
                                sh '''
                           npm install serve
                           node_modules/.bin/serve -s build &
                           sleep 10
                           npx playwright test --reporter=html
                        '''
                    }

                     post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PlayWright', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

         stage('Deploy') {
            agent {
                docker {
                    image 'node:18.18.2-alpine'
                    args '--network=host'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    netlify --version
                '''
            }
        }
    }
}