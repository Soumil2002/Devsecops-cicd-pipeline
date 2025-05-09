pipeline {
    agent any

    environment {
        IMAGE_NAME = "devsecops-flask-app"
        TAG = "latest"
        ECR_REGISTRY = "your_ecr_url" // if pushing to ECR
    }

    stages {
        stage('Clone') {
            steps {
                git 'https://github.com/<your-username>/devsecops-cicd-pipeline.git'
            }
        }

        stage('Code Quality - SonarQube') {
            steps {
                script {
                    withSonarQubeEnv('MySonarQube') {
                        sh "sonar-scanner"
                    }
                }
            }
        }

        stage('Security Scan - Trivy') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$TAG .'
                sh 'trivy image $IMAGE_NAME:$TAG || true'
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$TAG .'
                // Optional: Push to DockerHub or ECR
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml'
                sh 'kubectl apply -f k8s/service.yaml'
            }
        }
    }
}
