pipeline {
   agent any

   environment {
        NETLIFY_SITE_ID='79164785-f901-466a-9d77-639ed73d4016'
        NETLIFY_AUTH_TOKEN= credentials('netlify-token')
        REACT_APP_VERSION="1.2.${BUILD_ID}"
    }

    stages {
        stage('Docker') {
            steps {
                sh 'docker build -t my-playwright --network=host .'
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
                            image 'my-playwright:latest'
                            args '--network=host'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                           serve -s build &
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

//          stage('Deploy Staging') {
//             agent {
//                 docker {
//                     image 'node:18.18.2-alpine'
//                     args '--network=host'
//                     reuseNode true
//                 }
//             }
//             steps {
//                 sh '''
//                    npm install netlify-cli node-jq
//                    node_modules/.bin/netlify --version
//                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
//                    node_modules/.bin/netlify status
//                    node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
//                 '''
//
//                  script {
//                     env.STAGING_URL=sh(script:"node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
//                  }
//             }
//         }

         stage ('Deploy Stage & E2E') {
                    agent {
                        docker {
                            //image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            image 'my-playwright:latest'
                            args '--network=host'
                            reuseNode true
                        }
                    }

                    environment {
                        CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
                    }


                    //npm install netlify-cli node-jq
                    steps {
                        sh '''
                           netlify --version
                           echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                           netlify status
                           netlify deploy --dir=build --json > deploy-output.json
                           CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                           npx playwright test --reporter=html
                        '''
                    }

                     post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PlayWright E2E Staging Report Html', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }

         stage ('Deploy Prod & E2E') {
            agent {
                docker {
                    image 'my-playwright:latest'
                    args '--network=host'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://rococo-taiyaki-a47e12.netlify.app'
            }


            steps {
                sh '''
                 netlify --version
                 echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                 netlify status
                 netlify deploy --dir=build --prod
                 npx playwright test --reporter=html
                '''
            }

             post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PlayWright PROD E2E Report Html', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}