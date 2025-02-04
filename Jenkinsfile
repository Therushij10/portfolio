pipeline {
    agent any

    environment {
        IMAGE_NAME = "therushij10/my-portfolio:latest"  // Your Docker Hub image
        EC2_USER = "ubuntu"  // Change to 'ec2-user' if using Amazon Linux
        EC2_HOST = "13.233.190.129"  // Your EC2 Public IP
        SSH_CREDENTIALS_ID = "aws-key"  // Jenkins Credentials ID for SSH
        DOCKER_HUB_CREDENTIALS = "docker-hub-credentials"  // Jenkins Credentials ID for Docker Hub
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'master', url: 'https://github.com/Therushij10/portfolio.git'
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKER_HUB_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                        docker build -t ${IMAGE_NAME} .
                        echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
                        docker push ${IMAGE_NAME}
                        """
                    }
                }
            }
        }

        stage('Deploy to AWS EC2') {
            steps {
                sshagent([SSH_CREDENTIALS_ID]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << EOF
                        docker stop my-portfolio || true
                        docker rm my-portfolio || true
                        docker pull ${IMAGE_NAME}
                        docker run -d -p 3000:3000 --name my-portfolio ${IMAGE_NAME}
                    EOF
                    """
                }
            }
        }
    }
}
