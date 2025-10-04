pipeline {
    agent any
    
    environment {
        RREGISTRY = 'Dockerhub'
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

        stage('Build & Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        def app = docker.build("${REGISTRY}/${IMAGE}:${BUILD_NUMBER}")
                        app.push()
                        app.push("${IMAGE_TAG}")
                    }
                }
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
