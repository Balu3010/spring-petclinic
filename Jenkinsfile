pipeline {
    agent any

    environment {
        CONTAINER_NAME = "spring-petclinic"
        IMAGE_NAME = "balubojja/spring-petclinic"
        TAG = "latest"
        PORT = "8080"
        EC2_IP = "16.176.194.233"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/Balu3010/spring-petclinic.git'
            }
        }

        stage('Build JAR using Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$TAG .'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
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
}
