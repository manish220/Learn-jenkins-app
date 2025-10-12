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
                    npm ci
                    npx playwright install --with-deps
                    npx playwright --version
                    npx http-server build -p 3000 &
                    npx playwright --version
                    sleep 10
                    npx playwright test
                    npx playwright --version
                   
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