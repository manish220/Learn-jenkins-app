pipeline {
    agent any

    stages {
        stage('w/o docker') {
            steps {
                sh 'echo "without docker"'
            }
        }
        stage('w docker') {
            agent {
                docker {
                    image 'node:18-alpine'
                }
            }
            steps {
                sh 'echo "with docker"'
                sh 'npm --version'
            }
        }
    }
}
