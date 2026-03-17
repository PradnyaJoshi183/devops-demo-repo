pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        ACCOUNT_ID = "277528343142"
        ECR_REPO = "277528343142.dkr.ecr.ap-south-1.amazonaws.com/demo-app"
        TARGET_IP = "13.202.101.183"
    }

    stages {

        stage('Build Image') {
            steps {
                sh 'docker build -t demo-app .'
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com
                '''
            }
        }

        stage('Tag Image') {
            steps {
                sh 'docker tag demo-app:latest $ECR_REPO:latest'
            }
        }

        stage('Push Image') {
            steps {
                sh 'docker push $ECR_REPO:latest'
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['server-key']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@$TARGET_IP "
                    aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com
                    docker pull $ECR_REPO:latest
                    docker stop demo || true
                    docker rm demo || true
                    docker run -d -p 80:80 --name demo $ECR_REPO:latest
                    "
                    '''
                }
            }
        }
    }
}
