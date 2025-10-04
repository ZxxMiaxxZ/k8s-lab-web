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
                echo "Code đã được pull từ Git"
            }
        }
        stage('Check thu') {
            steps {
                sh "pwd"
                sh "ls"
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                    sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
                }
            }
        }
        stage('Debug before build') {
            steps {
                sh """
                    echo "Current dir: $(pwd)"
                    ls -l
                """
            }
        }

        
        stage('Deploy to K8s') {
            steps {
                script {
                    sh """
                        kubectl set image deployment/web-app web-app=${IMAGE_NAME}:${IMAGE_TAG} || \
                        kubectl create deployment web-app --image=${IMAGE_NAME}:${IMAGE_TAG} --replicas=3
                        
                        kubectl expose deployment web-app --type=NodePort --port=80 || true
                        kubectl rollout status deployment/web-app
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo "✅ Deploy thành công!"
            sh "kubectl get pods -l app=web-app"
        }
        failure {
            echo "❌ Deploy thất bại!"
        }
    }
}
