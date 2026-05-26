pipeline {
    agent any

    environment {
        IMAGE = "pragatipatil1610/hostel-app"
        EC2_USER = "ubuntu"
        EC2_HOST = "43.206.1.154"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/pragatipatil1610/Devops.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${IMAGE}:${BUILD_NUMBER} .
                    docker tag ${IMAGE}:${BUILD_NUMBER} ${IMAGE}:latest
                """
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_PASS'
                )]) {
                    sh """
                        echo $DH_PASS | docker login -u $DH_USER --password-stdin
                        docker push ${IMAGE}:${BUILD_NUMBER}
                        docker push ${IMAGE}:latest
                    """
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'ec2-ssh-key',
                        keyFileVariable: 'SSH_KEY'
                    )
                ]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no -i $SSH_KEY ubuntu@${EC2_HOST} '
                            docker pull ${IMAGE}:latest

                            docker stop hostel-app || true
                            docker rm hostel-app || true

                            docker run -d --name hostel-app \
                                -p 80:80 \
                                --restart always \
                                ${IMAGE}:latest
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "SUCCESS: Deployed automatically"
        }
        failure {
            echo "FAILED pipeline"
        }
    }
}
