pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1' // replace with your region
        ECR_REPO = 'public.ecr.aws/v7f8p0w8/cicd' // e.g. public.ecr.aws/abc123/my-app
        IMAGE_TAG = "latest"
    }

    stages {
        stage('Clone Repo') {
            steps {
                checkout scm
            }
        }

        stage('Generate Dockerfile') {
            steps {
                script {
                    writeFile file: 'Dockerfile', text: """
                    FROM nginx:alpine
                    COPY index.html /usr/share/nginx/html/index.html
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $ECR_REPO:$IMAGE_TAG .'
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr-public get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin public.ecr.aws
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh 'docker push $ECR_REPO:$IMAGE_TAG'
            }
        }
    }

    post {
        success {
            echo '✅ Image built and pushed to ECR successfully.'
        }
        failure {
            echo '❌ Build failed.'
        }
    }
}
