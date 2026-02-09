pipeline {
    agent any
 
    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "user-service"
        ECS_CLUSTER = "dev_cluster"
        ECS_SERVICE = "user-service-task-service-1t1xpq3y"
        IMAGE_TAG = "${BUILD_NUMBER}"
        AWS_ACCOUNT_ID = "600696639588"
        ECR_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
    }
 
    stages {
 
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
 
        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${ECR_REPO}:${IMAGE_TAG} .
                docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_URI}:${IMAGE_TAG}
                docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_URI}:latest
                """
            }
        }
 
        stage('Login to ECR') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh """
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    """
                }
            }
        }
 
        stage('Push to ECR') {
            steps {
                sh """
                docker push ${ECR_URI}:${IMAGE_TAG}
                docker push ${ECR_URI}:latest
                """
            }
        }
 
        stage('Deploy to ECS') {
            steps {
                sh """
                aws ecs update-service \
                    --cluster ${ECS_CLUSTER} \
                    --service ${ECS_SERVICE} \
                    --force-new-deployment \
                    --region ${AWS_REGION}
                """
            }
        }
    }
 
    post {
        always {
            sh "docker system prune -f"
        }
    }
}
