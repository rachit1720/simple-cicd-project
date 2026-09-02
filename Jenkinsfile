pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t simple-cicd-app .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                        -u "$DOCKER_USER" \
                        --password-stdin

                        docker tag simple-cicd-app:latest \
                        $DOCKER_USER/simple-cicd-project:latest

                        docker push \
                        $DOCKER_USER/simple-cicd-project:latest
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no \
                        ec2-user@13.204.64.166 "
                            cd /home/ec2-user/simple-cicd-deploy &&
                            docker compose pull &&
                            docker compose up -d
                        "
                    '''
                }
            }
        }
    }
}
