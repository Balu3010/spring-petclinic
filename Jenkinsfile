pipeline {
    agent any

    environment {
        IMAGE_NAME = "balubojja/spring-petclinic"
        EC2_IP = "18.205.245.129"
        PORT = "8080"
        TAG = "latest"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/Balu3010/spring-petclinic.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$TAG .'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push $IMAGE_NAME:$TAG'
            }
        }
        
        stage('Deploy on EC2 Instance') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@$EC2_IP '
                            docker pull $IMAGE_NAME:$TAG &&
                            docker stop $CONTAINER_NAME || true &&
                            docker rm $CONTAINER_NAME || true &&
                            docker run -d -p ${PORT}:${PORT} --name $CONTAINER_NAME $IMAGE_NAME:$TAG
                        '
                    '''
                }
            }
        }
    }    

    post {
        success {
            echo "✅ Docker image successfully built and pushed to Docker Hub!"
        }
        failure {
            echo "❌ Build or push failed. Check Jenkins logs for details."
        }
    }
}
