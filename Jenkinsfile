pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_USERNAME = 'jeeeiiiiiii'  // Replace with your actual username
        IMAGE_NAME = "${DOCKER_USERNAME}/aurjei-resume"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
            }
        }
        
        stage('Test') {
            steps {
                echo 'Testing stage - add your tests here'
            }
        }
        
        stage('Push to Registry') {
            steps {
                script {
                    // Use Jenkins credentials to login
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        // Login to Docker Hub
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        
                        // Push images
                        sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                        sh "docker push ${IMAGE_NAME}:latest"
                        
                        // Logout for security
                        sh 'docker logout'
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                sh 'kubectl apply -f kubernetes/service.yaml'
                sh "kubectl set image deployment/resume-deployment resume-container=${IMAGE_NAME}:${IMAGE_TAG}"
                sh 'kubectl rollout status deployment/resume-deployment --timeout=5m'
            }
        }
        
        stage('Cleanup') {
            steps {
                sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
                sh "docker rmi ${IMAGE_NAME}:latest || true"
            }
        }
    }
    
    post {
        failure {
            echo 'Pipeline failed! Rolling back...'
            sh 'kubectl rollout undo deployment/resume-deployment || true'
        }
        always {
            // Ensure logout even if pipeline fails
            sh 'docker logout || true'
        }
    }
}