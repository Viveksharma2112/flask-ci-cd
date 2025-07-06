pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/Viveksharma2112/flask-ci-cd'
        DOCKER_IMAGE = 'vivek20053/flask-app'
        DEPLOY_SERVER = 'ec2-user@13.203.202.22'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: "${REPO_URL}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t ${DOCKER_IMAGE}:latest .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    script {
                        sh '''
                            echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                            docker push ${DOCKER_IMAGE}:latest
                        '''
                    }
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                script {
                    sshagent(['deploy-key']) {
                        sh '''
                            ssh ${DEPLOY_SERVER} "docker stop flask-container || true && docker rm flask-container || true && docker pull ${DOCKER_IMAGE}:latest && docker run --name flask-container -d -p 5000:5000 ${DOCKER_IMAGE}:latest"
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }
    }
}
