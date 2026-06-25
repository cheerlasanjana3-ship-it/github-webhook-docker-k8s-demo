pipeline {
    agent any

    environment {
        IMAGE_NAME = 'cicd-webapp'
        IMAGE_TAG = "${BUILD_NUMBER}"
        K8S_DEPLOYMENT = 'cicd-webapp-deployment'
        K8S_SERVICE = 'cicd-webapp-service'
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Pulling latest code from GitHub'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image locally'
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
                sh 'docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest'
            }
        }

        stage('Load Image into Minikube') {
            steps {
                echo 'Loading image into Minikube (no Docker Hub needed)'
                sh 'minikube image load $IMAGE_NAME:latest'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying application to Minikube Kubernetes'
                sh 'kubectl apply -f deployment.yaml'
                sh 'kubectl apply -f service.yaml'
                echo 'Restarting deployment to pick up new image'
                sh 'kubectl rollout restart deployment/$K8S_DEPLOYMENT'
                sh 'kubectl rollout status deployment/$K8S_DEPLOYMENT'
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'kubectl get deployment'
                sh 'kubectl get pods -o wide'
                sh 'kubectl get svc'
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline Completed Successfully'
            echo 'Application URL: run "minikube service cicd-webapp-service --url"'
        }
        failure {
            echo 'CI/CD Pipeline Failed. Check Console Output.'
        }
    }
}
