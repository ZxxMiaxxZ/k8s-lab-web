pipeline {
    agent any
    
    environment {
        REGISTRY = 'manhduynguyen'        // username DockerHub, KHÔNG phải email
        IMAGE = 'web-app'
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
                    docker.withRegistry('https://registry.hub.docker.com', 'Dockerhub') { 
                        def app = docker.build("${REGISTRY}/${IMAGE}:${IMAGE_TAG}")
                        app.push()         // push theo build number
                        app.push("latest") // push thêm latest
                    }
                }
            }
        }
    }
    post {
        success {
            echo "✅ Build & Push thành công!"
        }
        failure {
            echo "❌ Deploy thất bại!"
        }
    }
}
