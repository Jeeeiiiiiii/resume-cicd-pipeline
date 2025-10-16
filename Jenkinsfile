pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_USERNAME = 'jeeeiiiiiii'
        IMAGE_NAME = "${DOCKER_USERNAME}/aurjei-resume"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} docker/"
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
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                        sh "docker push ${IMAGE_NAME}:latest"
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                dir('ansible') {
                    sh """
                        ansible-playbook playbooks/deploy-k8s.yml \
                        -e IMAGE_NAME=${IMAGE_NAME} \
                        -e IMAGE_TAG=${IMAGE_TAG}
                    """
                }
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
            // No withCredentials needed
            sh 'kubectl rollout undo deployment/resume-test || true'
        }
        always {
            sh 'docker logout || true'
        }
    }
}