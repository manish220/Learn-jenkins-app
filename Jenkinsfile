pipeline {
    agent any

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
                    npm run build
                    ls -la
                '''
            }
        }
        
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    # For Jest
                    npm test 
                   
                '''
            }
        }
        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.49.0-jammy'
                    reuseNode true
                }
            }
            steps {
                sh '''
                   
                   npx http-server build -p 3000 &
                   sleep 10
                   npx playwright test
                   
                '''
            }
        }
    } 
    
    post{
        always{
            junit 'jest-results/junit.xml'
        }
    }
}