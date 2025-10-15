pipeline {
    agent any

    
    environment {
        NODE_IMAGE = 'node:18-bullseye' // Debian-based, includes build tools
    }

    stages {

        stage('Prepare') {
            agent {
                docker {
                    image "${NODE_IMAGE}"
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Cleaning old dependencies..."
                    rm -rf node_modules package-lock.json
                    echo "Listing files before install:"
                    ls -la
                '''
            }
        }

        stage('Install Dependencies') {
            agent {
                docker {
                    image "${NODE_IMAGE}"
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Installing dependencies..."
                    npm ci  # deterministic install using package-lock.json
                '''
            }
        }

        stage('Build') {
            agent {
                docker {
                    image "${NODE_IMAGE}"
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Node & NPM versions:"
                    node --version
                    npm --version

                    echo "Building project..."
                    npm run build
                '''
            }
        }

        stage('Test') {
            agent {
                docker {
                    image "${NODE_IMAGE}"
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Running tests..."
                    npm test
                '''
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image "${NODE_IMAGE}"
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Installing Netlify CLI locally..."
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                '''
            }
        }

    }

    post {
        always {
            echo 'Publishing test results...'
            junit 'jest-results/junit.xml'
            cleanWs()
        }
    }
}
