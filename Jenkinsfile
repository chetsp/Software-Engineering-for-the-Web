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
                        # Using the static file to avoid YAML formatting errors
                        export KUBECONFIG=./k8s-config.yaml
                        
                        kubectl set image deployment/${DEPLOY_NAME} \
                          campusconnect=${DOCKER_USER}/${IMAGE_NAME}:${TIMESTAMP} \
                          -n ${K8S_NAMESPACE} \
                          --server=https://107.22.248.10:6443 \
                          --insecure-skip-tls-verify
                    """
                }
            }
        }
    } // <--- This was the missing brace for 'stages'

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Pipeline failed. Check the logs above.'
        }
    }
}
