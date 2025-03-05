pipeline {
   agent any

   environment {
        NETLIFY_SITE_ID='79164785-f901-466a-9d77-639ed73d4016'
        NETLIFY_AUTH_TOKEN= credentials('netlify-token')
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

         stage('Deploy Staging') {
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
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build
                '''
            }
        }

         stage('Approval') {
            steps {
                input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
            }
         }

         stage('Deploy Prod') {
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
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }

         stage ('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    args '--network=host'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://rococo-taiyaki-a47e12.netlify.app'
            }


            steps {
                sh '''
                   npx playwright test --reporter=html
                '''
            }

             post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PlayWright PROD Report Html', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}