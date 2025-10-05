pipeline {
    agent any
    
    environment {
        // Docker Hub info
        REGISTRY = 'manhduynguyen'
        IMAGE = 'web-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_CREDS = credentials('dockerhub')
        
        // Kubernetes info
        K8S_NAMESPACE = 'default'  // Hoặc 'dev', 'production', etc.
    }
    
    stages {
        stage('1. Checkout Code') {
            steps {
                checkout scm
                echo "✅ Code đã được checkout từ Git"
            }
        }
        
        stage('2. Build Docker Image') {
            steps {
                script {
                    echo "🔨 Building Docker image: ${REGISTRY}/${IMAGE}:${IMAGE_TAG}"
                    sh """
                        docker build -t ${REGISTRY}/${IMAGE}:${IMAGE_TAG} .
                        docker tag ${REGISTRY}/${IMAGE}:${IMAGE_TAG} ${REGISTRY}/${IMAGE}:latest
                        docker images | grep ${IMAGE}
                    """
                }
            }
        }
        
        stage('3. Push to Docker Hub') {
            steps {
                script {
                    echo "🚀 Pushing image to Docker Hub..."
                    sh '''
                        echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin
                        docker push ${REGISTRY}/${IMAGE}:${IMAGE_TAG}
                        docker push ${REGISTRY}/${IMAGE}:latest
                        docker logout
                    '''
                    echo "✅ Image pushed: ${REGISTRY}/${IMAGE}:${IMAGE_TAG}"
                }
            }
        }
        
        stage('4. Update K8s Deployment File') {
            steps {
                script {
                    echo "📝 Updating deployment.yaml with new image tag..."
                    sh """
                        # Backup original file
                        cp k8s/deployment.yaml k8s/deployment.yaml.bak
                        
                        # Update image tag in deployment.yaml
                        sed -i 's|image: ${REGISTRY}/${IMAGE}:.*|image: ${REGISTRY}/${IMAGE}:${IMAGE_TAG}|g' k8s/deployment.yaml
                        
                        # Verify the change
                        echo "=== Updated deployment.yaml ==="
                        cat k8s/deployment.yaml | grep -A 2 'image:'
                    """
                }
            }
        }
        
        stage('5. Deploy to Kubernetes') {
            steps {
                script {
                    echo "☸️  Deploying to Kubernetes cluster..."
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                        sh """
                            # Apply Service first (if not exists)
                            kubectl --kubeconfig=\$KUBECONFIG apply -f k8s/service.yaml -n ${K8S_NAMESPACE}
                            
                            # Apply Deployment
                            kubectl --kubeconfig=\$KUBECONFIG apply -f k8s/deployment.yaml -n ${K8S_NAMESPACE}
                            
                            # Wait for rollout to complete (timeout 5 minutes)
                            kubectl --kubeconfig=\$KUBECONFIG rollout status deployment/web-app -n ${K8S_NAMESPACE} --timeout=5m
                        """
                    }
                    echo "✅ Deployment completed successfully"
                }
            }
        }
        
        stage('6. Verify Deployment') {
            steps {
                script {
                    echo "🔍 Verifying deployment..."
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                        sh """
                            echo "=== DEPLOYMENTS ==="
                            kubectl --kubeconfig=\$KUBECONFIG get deployments -n ${K8S_NAMESPACE}
                            
                            echo ""
                            echo "=== PODS ==="
                            kubectl --kubeconfig=\$KUBECONFIG get pods -l app=web-app -n ${K8S_NAMESPACE}
                            
                            echo ""
                            echo "=== SERVICES ==="
                            kubectl --kubeconfig=\$KUBECONFIG get svc -n ${K8S_NAMESPACE}
                            
                            echo ""
                            echo "=== SERVICE URL ==="
                            kubectl --kubeconfig=\$KUBECONFIG get svc web-app-service -n ${K8S_NAMESPACE} -o wide
                        """
                    }
                }
            }
        }
        
        stage('7. Cleanup Local Images') {
            steps {
                script {
                    echo "🧹 Cleaning up local Docker images..."
                    sh """
                        docker rmi ${REGISTRY}/${IMAGE}:${IMAGE_TAG} || true
                        docker rmi ${REGISTRY}/${IMAGE}:latest || true
                    """
                }
            }
        }
    }
    
    post {
        success {
            script {
                echo """
                ═══════════════════════════════════════════
                ✅ DEPLOYMENT SUCCESSFUL!
                ═══════════════════════════════════════════
                🐳 Docker Image: ${REGISTRY}/${IMAGE}:${IMAGE_TAG}
                📦 Docker Hub: https://hub.docker.com/r/${REGISTRY}/${IMAGE}
                ☸️  Namespace: ${K8S_NAMESPACE}
                🔢 Build Number: ${BUILD_NUMBER}
                ═══════════════════════════════════════════
                """
            }
        }
        failure {
            echo """
            ❌ DEPLOYMENT FAILED!
            Please check the logs above for errors.
            """
        }
        always {
            sh 'docker logout || true'
        }
    }
}
