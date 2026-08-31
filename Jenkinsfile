pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "tushar25l/devops-react-app"
        DOCKER_TAG = "${BUILD_NUMBER}"
        K8S_SERVER = "ubuntu@172.31.13.155"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build React App') {
            steps {
                sh 'npm ci'
                sh 'npm run build'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    ssh -o StrictHostKeyChecking=no ${K8S_SERVER} '
                        kubectl set image deployment/reactapp-deployment \
                        reactapp=${DOCKER_IMAGE}:${DOCKER_TAG}

                        kubectl rollout status deployment/reactapp-deployment
                    '
                """
            }
        }
    }
}
