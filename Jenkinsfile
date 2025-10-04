pipeline {
    agent any
    
    environment {
        IMAGE_NAME = 'web-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "✅ Code đã được pull từ Git"
            }
        }

        stage('Check thu') {
            steps {
                sh "pwd"
                sh "ls -l"
            }
        }

        stage('Debug before build') {
            steps {
                sh '''
                    echo "📂 Current dir: $(pwd)"
                    ls -l
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        cd ${WORKSPACE}
                        ls -l
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                    """
                }
            }
        }

    post {
        success {
            echo "thành công!"
        }
        failure {
            echo "❌ Deploy thất bại!"
        }
    }
}
