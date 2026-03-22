pipeline {
    agent any

    environment {
        ECR_REPO = "<AWS_ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/demo-app"
        REGION   = "ap-south-1"
        EC2_USER = "ec2-user"
        EC2_IP   = "<EC2-PUBLIC-IP>"
        PEM_FILE = "/var/lib/jenkins/test99.pem"
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main', url: 'https://github.com/cloud-guide/static-website-repo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t demo-app ./app'
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $ECR_REPO
                '''
            }
        }

        stage('Tag & Push Image') {
            steps {
                sh '''
                docker tag demo-app:latest $ECR_REPO:latest
                docker push $ECR_REPO:latest
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no -i $PEM_FILE $EC2_USER@$EC2_IP "
                docker pull $ECR_REPO:latest &&
                docker stop $(docker ps -q --filter ancestor=$ECR_REPO) || true &&
                docker run -d -p 5000:5000 $ECR_REPO:latest
                "
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully! App deployed.'
        }
        failure {
            echo '❌ Pipeline failed. Check logs.'
        }
    }
}
