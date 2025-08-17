pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        FRONTEND_ECR_REPO = '247329439643.dkr.ecr.ap-south-1.amazonaws.com/my-app-frontend'
        BACKEND_ECR_REPO  = '247329439643.dkr.ecr.ap-south-1.amazonaws.com/my-app-backend'
        IMAGE_TAG = "latest"
    }

    stages {
        stage('Build Docker Images') {
            steps {
                script {
                    dir('frontend') {
                        sh """
                            aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $FRONTEND_ECR_REPO
                            docker build --no-cache -t $FRONTEND_ECR_REPO:$IMAGE_TAG .
                        """
                    }
                    dir('backend') {
                        sh """
                            aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $BACKEND_ECR_REPO
                            docker build --no-cache -t $BACKEND_ECR_REPO:$IMAGE_TAG .
                        """
                    }
                }
            }
        }

        stage('Push Docker Images to ECR') {
            steps {
                script {
                    sh "docker push $FRONTEND_ECR_REPO:$IMAGE_TAG"
                    sh "docker push $BACKEND_ECR_REPO:$IMAGE_TAG"
                }
            }
        }

        stage('Deploy Containers on Server') {
            steps {
                script {
                    sh """
                        docker ps -q --filter 'name=my-app-frontend' | grep -q . && docker stop my-app-frontend && docker rm my-app-frontend || true
                        docker run -d --name my-app-frontend -p 8080:8080 $FRONTEND_ECR_REPO:$IMAGE_TAG
                        docker ps -q --filter 'name=my-app-backend' | grep -q . && docker stop my-app-backend && docker rm my-app-backend || true
                        docker run -d --name my-app-backend -p 5000:5000 $BACKEND_ECR_REPO:$IMAGE_TAG
                    """
                }
            }
        }
    }
}
