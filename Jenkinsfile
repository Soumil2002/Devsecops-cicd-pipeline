pipeline {
    agent any

    tools {
        sonarQubeScanner 'SonarScanner'  
    }

    environment {
        SCANNER_HOME = tool 'SonarScanner'
        DOCKER_IMAGE_NAME = 'devsecops-flask-app'
        DOCKER_HUB_REPO = 'soumil22/devsecops-flask-app'
        KUBERNETES_CLUSTER_NAME = 'devsecops-cluster'
        KUBERNETES_NAMESPACE = 'default'
    }

    stages {
        // Stage 1: Checkout Code from GitHub
        stage('Checkout') {
            steps {
                git 'https://github.com/Soumil2002/Devsecops-cicd-pipeline.git'
            }
        }

        // Stage 2: Run SonarQube Code Analysis
        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('MySonarQube') {
                        sh "${SCANNER_HOME}/bin/sonar-scanner"
                    }
                }
            }
        }

        // Stage 3: Build Docker Image
        stage('Docker Build') {
            steps {
                script {
                    sh 'docker build -t ${DOCKER_IMAGE_NAME} .'
                }
            }
        }

        // Stage 4: Trivy Vulnerability Scan
        stage('Trivy Scan') {
            steps {
                script {
                    sh 'trivy image --severity HIGH,CRITICAL ${DOCKER_IMAGE_NAME}'
                }
            }
        }

        // Stage 5: Push Docker Image to Docker Hub
        stage('Docker Push to Docker Hub') {
            steps {
                script {
                    // Login to Docker Hub using the credentials stored in Jenkins
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
                    }

                    // Tag Docker image for Docker Hub
                    sh 'docker tag ${DOCKER_IMAGE_NAME}:latest $DOCKER_HUB_REPO:latest'

                    // Push the Docker image to Docker Hub
                    sh 'docker push $DOCKER_HUB_REPO:latest'
                }
            }
        }

        // Stage 6: Deploy to EKS using kubectl
        stage('Deploy to EKS') {
            steps {
                script {
                    // Apply Kubernetes manifests to EKS (Deployment and Service YAML)
                    sh 'kubectl apply -f deployment.yaml --namespace=$KUBERNETES_NAMESPACE'
                    sh 'kubectl apply -f service.yaml --namespace=$KUBERNETES_NAMESPACE'
                }
            }
        }
    }

    post {
        always {
            // Archive the SonarQube analysis results or logs if necessary
            archiveArtifacts artifacts: '**/sonar-report.json', allowEmptyArchive: true
        }

        success {
            echo 'Pipeline finished successfully!'
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}
