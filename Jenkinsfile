pipeline {
    agent any

    environment {
        DOCKERHUB_PASS = credentials('docker-hub-pass')
        DOCKER_USER = 'chetsp2613'
        IMAGE_NAME = 'campusconnect'
        K8S_NAMESPACE = 'campusconnect-ns'
        DEPLOY_NAME = 'campusconnect-deployment'
        TIMESTAMP = sh(script: "date +%Y%m%d%H%M%S", returnStdout: true).trim()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    sh "docker login -u ${DOCKER_USER} -p ${DOCKERHUB_PASS}"
                    sh "docker build -t ${DOCKER_USER}/${IMAGE_NAME}:${TIMESTAMP} ."
                    sh "docker push ${DOCKER_USER}/${IMAGE_NAME}:${TIMESTAMP}"
                    sh "docker tag ${DOCKER_USER}/${IMAGE_NAME}:${TIMESTAMP} ${DOCKER_USER}/${IMAGE_NAME}:latest"
                    sh "docker push ${DOCKER_USER}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                        kubectl set image deployment/${DEPLOY_NAME} \
                          campusconnect=${DOCKER_USER}/${IMAGE_NAME}:${TIMESTAMP} \
                          -n ${K8S_NAMESPACE} \
                          --server=https://107.22.248.10:6443 \
                          --token=K106578e08a6b08a709ad5950ae2f73981a26b47386c50a15ac70673f8dda03f2e8::server:jspk4q5s5cx9zr5fkpjx5p8wmbd8f6cfqpc5zmd2pnvkrlhmlmrd8k \
                          --insecure-skip-tls-verify
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Pipeline failed. Check the logs above.'
        }
    }
}
