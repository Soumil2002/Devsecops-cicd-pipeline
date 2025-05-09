pipeline {
    agent any

    environment {
        // SonarQube configuration (no need for tools block with newer plugin)
        SONAR_SCANNER_OPTS = "-Dsonar.projectName=DevSecOps-Flask-App"
        DOCKER_IMAGE_NAME = 'devsecops-flask-app'
        DOCKER_HUB_REPO = 'soumil22/devsecops-flask-app'
        KUBERNETES_CLUSTER_NAME = 'devsecops-cluster'
        KUBERNETES_NAMESPACE = 'default'
    }

    stages {
        // Stage 1: Checkout Code from GitHub
        stage('Checkout') {
            steps {
                git branch: 'main', 
                url: 'https://github.com/Soumil2002/Devsecops-cicd-pipeline.git'
            }
        }

        // Stage 2: Run SonarQube Code Analysis
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonarQube') {
                    // Using the default sonar-scanner from the plugin
                    sh 'sonar-scanner \
                        -Dsonar.projectKey=DevSecOps-Flask-App \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=${SONAR_AUTH_TOKEN}'
                }
            }
        }

        // Stage 3: Build Docker Image
        stage('Docker Build') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE_NAME} ."
                }
            }
        }

        // Stage 4: Trivy Vulnerability Scan
        stage('Trivy Scan') {
            steps {
                script {
                    // Exit if critical vulnerabilities found
                    sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL ${DOCKER_IMAGE_NAME}'
                }
            }
        }

        // Stage 5: Push Docker Image to Docker Hub
        stage('Docker Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-credentials', 
                        usernameVariable: 'DOCKER_USER', 
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh "docker login -u $DOCKER_USER -p $DOCKER_PASS"
                        sh "docker tag ${DOCKER_IMAGE_NAME}:latest ${DOCKER_HUB_REPO}:latest"
                        sh "docker push ${DOCKER_HUB_REPO}:latest"
                    }
                }
            }
        }

        // Stage 6: Deploy to EKS using kubectl
        stage('Deploy to EKS') {
            steps {
                script {
                    // Configure kubectl to use the right context
                    sh "kubectl config use-context ${KUBERNETES_CLUSTER_NAME}"
                    
                    // Apply Kubernetes manifests
                    sh "kubectl apply -f deployment.yaml -n ${KUBERNETES_NAMESPACE}"
                    sh "kubectl apply -f service.yaml -n ${KUBERNETES_NAMESPACE}"
                    
                    // Verify deployment
                    sh "kubectl rollout status deployment/devsecops-flask-app -n ${KUBERNETES_NAMESPACE}"
                }
            }
        }
    }

    post {
        always {
            // Clean up workspace
            cleanWs()
            
            // Send notifications (you'll need to configure email/chat plugins)
            script {
                if (currentBuild.result == 'SUCCESS') {
                    // Success notification
                } else {
                    // Failure notification
                }
            }
        }
    }
}
