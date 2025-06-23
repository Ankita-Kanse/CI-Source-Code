pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = 'public.ecr.aws/v7f8p0w8/cicd'
        IMAGE_TAG = "build-${BUILD_NUMBER}"
        DEPLOY_REPO = 'https://github.com/Ankita-Kanse/ArgoCD.git'
    }

    stages {
        stage('Clone Source Repo') {
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

        stage('Login to Public ECR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                    mkdir -p ~/.aws
                    echo "[default]" > ~/.aws/credentials
                    echo "aws_access_key_id=$AWS_ACCESS_KEY_ID" >> ~/.aws/credentials
                    echo "aws_secret_access_key=$AWS_SECRET_ACCESS_KEY" >> ~/.aws/credentials

                    aws ecr-public get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin public.ecr.aws
                    '''
                }
            }
        }

        stage('Push to ECR') {
            steps {
                sh 'docker push $ECR_REPO:$IMAGE_TAG'
            }
        }

        stage('Update Deployment YAML in Git') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'git-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                    sh '''
                    rm -rf ArgoCD
                    git clone https://$GIT_USER:$GIT_PASS@${DEPLOY_REPO#https://}
                    cd ArgoCD

                    git config user.email "jenkins@demo.com"
                    git config user.name "Jenkins"

                    git pull origin main
                    git reset --hard

                    sed -i "s|image: .*|image: public.ecr.aws/v7f8p0w8/cicd:$IMAGE_TAG|" k8s/deployment.yaml

                    git add k8s/deployment.yaml
                    git commit -m "CI: Update image tag to $IMAGE_TAG"
                    git push origin main
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ CI/CD Pipeline completed. Image pushed and ArgoCD will auto-deploy.'
        }
        failure {
            echo '❌ Build or deployment update failed.'
        }
    }
}
