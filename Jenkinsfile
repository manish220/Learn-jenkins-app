pipeline {
    agent any

    environment {
        BUILD_DIR = 'build'
        INDEX_HTML = 'index.html'
        NETLIFY_SITE_ID = '34e69c4e-c9f9-4ebc-9b60-bdce1d2ab778'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }
 
    stages {
        // this is a comment
        /*
            this is a block comment
            second line
        */
 
        stage('Build') {
            agent {
                docker {
                    image 'my-docker'
                    reuseNode true
                }
            }
 /*           agent {
               docker {
                    image 'node:22-alpine'
                    reuseNode true
                }
            }
*/            steps {
                sh '''
                    ls -la
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

       stage ('AWS-CLI') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "--entrypoint=''"
                }
            }

            environment {
                AWS_BUCKET = 'learn-jenkins-202510141554'
            }
            
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        aws s3 ls
#                        echo "Hello S3!" > index.html
#                       aws s3 cp index.html s3://$AWS_BUCKET_NAME/index.html
#                      aws s3 sync . s3://$AWS_BUCKET_NAME/index.html    curruent directory
#                      aws s3 sync build s3://$AWS_BUCKET/index.html     wrong dirname
                      aws s3 sync build s3://$AWS_BUCKET    
                          aws s3 ls
                    '''
                }
            }
        }

        stage ('Run Tests') {
            parallel { 
                stage ('Unit Testing') {
                    agent {
                        docker {
                            image 'my-docker'
                            reuseNode true
                        }
                    }
/*                    agent {
                        docker {
                            image 'node:22-alpine'
                            reuseNode true
                        }
                    }
*/
                    steps {
                        echo 'Testing the app ...'
                        sh '''
                            #test -f $BUILD_DIR/'index.html'  this is of course a linux comment
                            npm test
                        '''
                    }

                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage ('End2End Testing') {
                    agent {
                        docker {
                            image 'my-docker'
                            reuseNode true
                            //args '-u root:root' !!! don't do this!!  TO SPECIFY ANOTHER USER & GROUP
                        }
                    }

                    steps {
                        echo 'Testing the app ...'
                        sh '''
                            serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                           publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Local E2E - HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

       stage ('Staging Deployment & E2E Testing') {
            agent {
                docker {
                    image 'my-docker'
                    reuseNode true
                    //args '-u root:root' !!! don't do this!!  TO SPECIFY ANOTHER USER & GROUP
                }
            }
            
            environment {
                 CI_ENVIRONMENT_URL='STAGING_URL_TO_BE_SET'
            }

            steps {
                echo 'Testing the app in production ...'
                sh '''
                    netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --no-build --json > staging-output.json
                    CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' staging-output.json)
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'STAGING E2E - HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

/*      Bu approval stage ile continious indelivery yapıyor idik. Bunu kaldırarak continious deployment'a geçmiş oluyoruz
        stage ('Approval') {
           steps {
                 timeout(time: 1, unit: 'MINUTES') {
                    input message: 'Ready to Deploy?', ok: 'Yes, I am sure I want to deploy!', cancel: 'No, under no circumstances!'
                }
            }
        }
*/

        stage ('Prod Deploction deployment & E2E Testing') {
            agent {
                docker {
                    image 'my-docker'
                    reuseNode true
                    //args '-u root:root' !!! don't do this!!  TO SPECIFY ANOTHER USER & GROUP
                }
            }
            
            environment {
                 CI_ENVIRONMENT_URL='https://prod-somsom.netlify.app'
            }

            steps {
                echo 'Testing the app in production ...'
                sh '''
                    netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    
                    netlify deploy --dir=build --no-build --prod 
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PROD E2E - HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

     }
}