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
                    echo "Cleaning old dependencies..."
                    rm -rf node_modules package-lock.json

                    echo "Listing files before install:"
                    ls -la

                    echo "Installing dependencies..."
                    npm install

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

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                   npm install netlify-cli
                   node_modules/.bin/netlify --version
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