pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'pragatipatil1610'
        IMAGE = "pragatipatil1610/hostel-app"
        EC2_USER = 'ubuntu'
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
                sh '''
                    docker build -t $IMAGE:$BUILD_NUMBER \
                                 -t $IMAGE:latest .
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_PASS'
                )]) {
                    sh '''
                        echo $DH_PASS | docker login -u $DH_USER --password-stdin

                        docker push $IMAGE:$BUILD_NUMBER
                        docker push $IMAGE:latest
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'ec2-ssh-key',
                        keyFileVariable: 'SSH_KEY'
                    ),
                    string(
                        credentialsId: 'ec2-host',
                        variable: 'EC2_HOST'
                    ),
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DH_USER',
                        passwordVariable: 'DH_PASS'
                    )
                ]) {

                    sh '''
                        ssh -o StrictHostKeyChecking=no -i $SSH_KEY $EC2_USER@$EC2_HOST "
                            docker login -u $DH_USER -p $DH_PASS

                            docker pull $IMAGE:latest

                            docker stop hostel-app || true
                            docker rm hostel-app || true

                            docker run -d --name hostel-app -p 80:80 --restart always $IMAGE:latest
                        "
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment SUCCESS — Build #${BUILD_NUMBER}"
        }

        failure {
            echo "❌ Pipeline FAILED — check logs"
        }

        always {
            sh 'docker logout || true'
        }
    }
}
