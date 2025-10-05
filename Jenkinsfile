pipeline {
    agent any
    
    environment {
        REGISTRY = 'manhduynguyen'
        IMAGE = 'web-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
        K8S_NAMESPACE = 'appsec'  // Thay đổi nếu cần: dev, staging, production
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
                    echo "✅ Image pushed to Docker Hub: ${REGISTRY}/${IMAGE}:${IMAGE_TAG}"
                }
            }
        }
        
        stage('Update K8s Manifest') {
            steps {
                script {
                    echo "📝 Updating deployment.yaml với image mới..."
                    sh """
                        # Backup file gốc
                        cp k8s/deployment.yaml k8s/deployment.yaml.bak
                        
                        # Update image tag
                        sed -i 's|image: ${REGISTRY}/${IMAGE}:.*|image: ${REGISTRY}/${IMAGE}:${IMAGE_TAG}|g' k8s/deployment.yaml
                        
                        # Show kết quả
                        echo "=== Image đã được update ==="
                        cat k8s/deployment.yaml | grep -A 2 'image:'
                    """
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "☸️  Deploying to Kubernetes..."
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                        sh """
                            # Apply Service trước (nếu chưa có)
                            kubectl --kubeconfig=\$KUBECONFIG apply -f k8s/service.yaml -n ${K8S_NAMESPACE}
                            
                            # Apply Deployment
                            kubectl --kubeconfig=\$KUBECONFIG apply -f k8s/deployment.yaml -n ${K8S_NAMESPACE}
                            
                            # Đợi deployment hoàn thành
                            kubectl --kubeconfig=\$KUBECONFIG rollout status deployment/web-app -n ${K8S_NAMESPACE} --timeout=5m
                        """
                    }
                    echo "✅ Deploy thành công!"
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    echo "🔍 Checking deployment status..."
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                        sh """
                            echo "=== PODS ==="
                            kubectl --kubeconfig=\$KUBECONFIG get pods -l app=web-app -n ${K8S_NAMESPACE}
                            
                            echo ""
                            echo "=== SERVICE ==="
                            kubectl --kubeconfig=\$KUBECONFIG get svc web-app-service -n ${K8S_NAMESPACE}
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo """
            ═══════════════════════════════════════
            ✅ BUILD & DEPLOY THÀNH CÔNG!
            ═══════════════════════════════════════
            🐳 Image: ${REGISTRY}/${IMAGE}:${IMAGE_TAG}
            🔗 Hub: https://hub.docker.com/r/${REGISTRY}/${IMAGE}
            ☸️  K8s Namespace: ${K8S_NAMESPACE}
            ═══════════════════════════════════════
            """
        }
        failure {
            echo "❌ Deploy thất bại! Kiểm tra logs phía trên"
        }
    }
}
