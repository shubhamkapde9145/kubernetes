pipeline {
    agent any

    environment {
        APP_NAME = "my-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_REPO = "your-dockerhub-username/my-app"
        KUBE_NAMESPACE = "default"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/your-username/your-repo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${DOCKER_REPO}:${IMAGE_TAG} .
                    docker tag ${DOCKER_REPO}:${IMAGE_TAG} ${DOCKER_REPO}:latest
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push '${DOCKER_REPO}:${IMAGE_TAG}'
                        docker push '${DOCKER_REPO}:latest'
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    kubectl set image deployment/${APP_NAME} \
                    ${APP_NAME}=${DOCKER_REPO}:${IMAGE_TAG} \
                    -n ${KUBE_NAMESPACE}

                    kubectl rollout status deployment/${APP_NAME} \
                    -n ${KUBE_NAMESPACE}
                """
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful'
        }
        failure {
            echo 'Deployment Failed'
        }
    }
}
