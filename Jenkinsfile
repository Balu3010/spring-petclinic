pipeline {
    agent any

    environment {
        CONTAINER_NAME = "balubojja/spring-petclinic"
        IMAGE_NAME = "spring-petclinic"
        TAG = "latest"
        EMAIL = "devops.balu3010@gmail.com"
        PORT = "8080"
        EC2_IP = "18.205.245.129"
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

        stage('Send Email Notification') {
            steps {
                emailtext(
                    subject: "✅ Spring Petclinic Deployed Successfully!",
                    body: """
                    Hello Team 👋,
                    \nYour Spring Petclinic app has been successfully deployed on EC2!
                    \n🌍 URL: http://${EC2_IP}:${PORT}/
                    \n🐳 Docker Image: ${IMAGE_NAME}:${TAG}
                    """,
                    to: "${devops.balu3010@gmail.com}"
                )
            }
        }
    }

    post {
        failure {
            emailtext(
                subject: "❌ Spring Petclinic Deployment Failed!",
                body: """
                Hello,
                \nDeployment failed for Spring Petclinic.
                \nPlease check Jenkins logs for error details.
                """,
                to: "${devops.balu3010@gmail.com}"
            )
        }
    }
}

