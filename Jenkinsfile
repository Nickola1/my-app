pipeline {
    agent any

    environment {
        APP_NAME = 'my-app'
        DOCKER_IMAGE = nickola1331/${APP_NAME}:${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                        sh "docker build -t \${DOCKER_IMAGE} ./app"
                        sh "docker push \${DOCKER_IMAGE}"
                        sh "docker logout"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes via Helm') {
            steps {
                sh """
                    helm upgrade --install \${APP_NAME} ./helm/\${APP_NAME} \\
                        --namespace \${APP_NAME} --create-namespace \\
                        --set image.tag=\${BUILD_NUMBER}
                """
            }
        }
    }

    post {
        always {
            sh "docker image prune -f"
        }
    }
}
