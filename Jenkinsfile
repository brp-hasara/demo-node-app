pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'hasarartc97/demo-hasara-node-app'
        DOCKER_HUB_CREDENTIALS_ID = 'HASARA_DOCKER_HUB_CREDENTIALS_ID'
        SSH_CREDENTIALS_ID = 'HASARA_AWS_SSH_CREDENTIALS_ID'
        EC2_HOST = '54.210.66.151'
        EC2_USERNAME= 'ubuntu'
        APP_NAME = 'demo-hasara-node-app'
    }
    
    stages {
        stage('Clone repo') {
            steps {
                git url: 'https://github.com/brp-hasara/demo-node-app.git', branch: 'main'
                echo "Repo cloned!"
            }
        }
        stage('Build docker image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_HUB_REPO}:${env.BUILD_ID}")
                    echo "Docker image built!"
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_HUB_CREDENTIALS_ID}") {
                        dockerImage.push()
                    }
                    echo "Docker image pushed to Docker Hub!"
                }
            }
        }
        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ["${SSH_CREDENTIALS_ID}"]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${EC2_USERNAME}@${EC2_HOST} '
                    sudo docker pull ${DOCKER_HUB_REPO}:${env.BUILD_ID}
                    sudo docker stop ${APP_NAME} || true
                    sudo docker rm ${APP_NAME} || true
                    sudo docker run -d --name ${APP_NAME} -p 8081:5000 ${DOCKER_HUB_REPO}:${env.BUILD_ID}
                    '
                    """
                }
            }
        }
        
    }
}
