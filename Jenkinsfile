pipeline {
    agent any
    
    environment {
        AWS_REGION = 'ap-south-1'  // Change if different
        ECR_REPO = '253985439142.dkr.ecr.ap-south-1.amazonaws.com/my-web-app'  // Your ECR URI
        EC2_HOST = '13.233.130.114'  // Your EC2 IP
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Sindanika/my-cicd-project.git'
            }
        }
        
        stage('Build') {
            steps {
                script {
                    echo 'Building application...'
                    sh 'npm install'
                }
            }
        }
        
        stage('Test') {
            steps {
                script {
                    echo 'Running tests...'
                    sh 'npm test'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image...'
                    sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
                    sh "docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_REPO}:latest"
                }
            }
        }
        
        stage('Push to ECR') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                        string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
                    ]) {
                        sh """
                            aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}
                            docker push ${ECR_REPO}:${IMAGE_TAG}
                            docker push ${ECR_REPO}:latest
                        """
                    }
                }
            }
        }
        
        stage('Deploy to EC2') {
            steps {
                script {
                    sshagent(['ec2-ssh-key']) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ec2-user@${EC2_HOST} "
                                aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}
                                docker pull ${ECR_REPO}:latest
                                docker stop myapp || true
                                docker rm myapp || true
                                docker run -d --name myapp -p 80:3000 ${ECR_REPO}:latest
                            "
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
