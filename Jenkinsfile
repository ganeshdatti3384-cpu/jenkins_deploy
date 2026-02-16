pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "ganeshdatti76/jenkins-node-app"
        EC2_HOST = "ubuntu@3.110.162.76"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main',
                url: 'https://github.com/ganeshdatti3384-cpu/jenkins_deploy.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push $DOCKER_IMAGE'
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-key']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no $EC2_HOST "
                    docker pull $DOCKER_IMAGE &&
                    docker stop jenkins-node-app || true &&
                    docker rm jenkins-node-app || true &&
                    docker run -d -p 80:3000 --name jenkins-node-app $DOCKER_IMAGE
                    "
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment Successful!'
        }
        failure {
            echo '❌ Deployment Failed!'
        }
    }
}
