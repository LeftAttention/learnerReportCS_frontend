pipeline {
    agent any

    environment {
        ECR_URL = "${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.REGION}.amazonaws.com"
        NAMESPACE = 'default'
    }

    stages {
        stage('Build Backend Image') {
            steps {
                script {
                    def backendImage = docker.build("lrc-backend:latest")
                    backendImage.push("${ECR_URL}/lrc-backend:latest")
                }
            }
        }
        stage('Build Frontend Image') {
            steps {
                script {
                    def frontendImage = docker.build("lrc-frontend:latest")
                    frontendImage.push("${ECR_URL}/lrc-frontend:latest")
                }
            }
        }
        stage('Deploy Backend') {
            steps {
                script {
                    sh 'helm upgrade --install lrc-backend ./lrc-backend --namespace ${NAMESPACE}'
                }
            }
        }
        stage('Get Backend Service IP') {
            steps {
                script {
                    SERVICE_IP = sh(script: "kubectl get svc --namespace ${NAMESPACE} lrc-backend --template '{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}'", returnStdout: true).trim()
                }
            }
        }
        stage('Update Frontend Configuration') {
            steps {
                script {
                    sh "echo BACKEND_URL=http://${SERVICE_IP} > ./lrc-frontend/config.env"
                    def frontendImage = docker.build("lrc-frontend:latest") // Rebuild frontend with updated config
                    frontendImage.push("${ECR_URL}/lrc-frontend:latest")
                }
            }
        }
        stage('Deploy Frontend') {
            steps {
                script {
                    sh 'helm upgrade --install lrc-frontend ./lrc-frontend --namespace ${NAMESPACE}'
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully.'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}
