pipeline {
    agent any

    environment {
        NODE_IMAGE = 'node:18-bullseye' // Debian-based Node image with build tools
        NETLIFY_SITE_ID = '242cdea1-d3ec-4dff-9c3e-5fe8c2a39c5c'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {

        stage('Prepare Workspace') {
            agent {
                docker {
                    image "${NODE_IMAGE}"
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Cleaning workspace before build..."
                    rm -rf node_modules

                    echo "Listing files:"
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
                    npm install
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
                    echo "Node version: $(node --version)"
                    echo "NPM version: $(npm --version)"

                    echo "Building project..."
                    npm run build

                    echo "Listing files after build:"
                    ls -la
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
                    echo "Running tests."
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
                    echo "Netlify CLI version:"
                    node_modules/.bin/netlify --version

                    echo '$NETLIFY_PROJECT_ID'

                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --prod --dir=build --site=$NETLIFY_SITE_ID
                '''
            }
        }

    }

    post {
        always {
            echo "Publishing test results..."
            junit 'jest-results/junit.xml'

            echo "Cleaning workspace after build..."
            cleanWs()  // Deletes entire workspace to prevent leftovers
        }
    }
}
