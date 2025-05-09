pipeline {
    agent any

    environment {
        // SonarQube configuration
        SONAR_SCANNER_HOME = tool name: 'sonar-scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
        PATH = "${env.SONAR_SCANNER_HOME}/bin:${env.PATH}"
        
        // Other environment variables
        DOCKER_IMAGE_NAME = 'devsecops-flask-app'
        DOCKER_HUB_REPO = 'soumil22/devsecops-flask-app'
        KUBERNETES_CLUSTER_NAME = 'devsecops-cluster'
        KUBERNETES_NAMESPACE = 'default'
    }

    stages {
        // Stage 1: Checkout Code from GitHub
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', 
                         branches: [[name: '*/main']],
                         extensions: [], 
                         userRemoteConfigs: [[url: 'https://github.com/Soumil2002/Devsecops-cicd-pipeline.git']]
                        ])
            }
        }

        // Stage 2: Run SonarQube Code Analysis
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonarQube') {
                    sh """
                        sonar-scanner \
                        -Dsonar.projectKey=DevSecOps-Flask-App \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=\${SONAR_HOST_URL} \
                        -Dsonar.login=\${SONAR_AUTH_TOKEN}
                    """
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
                    sh "trivy image --exit-code 1 --severity HIGH,CRITICAL ${DOCKER_IMAGE_NAME}"
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
                        sh """
                            docker login -u $DOCKER_USER -p $DOCKER_PASS
                            docker tag ${DOCKER_IMAGE_NAME}:latest ${DOCKER_HUB_REPO}:latest
                            docker push ${DOCKER_HUB_REPO}:latest
                        """
                    }
                }
            }
        }

        // Stage 6: Deploy to EKS using kubectl
        stage('Deploy to EKS') {
            steps {
                script {
                    sh """
                        kubectl config use-context ${KUBERNETES_CLUSTER_NAME}
                        kubectl apply -f deployment.yaml -n ${KUBERNETES_NAMESPACE}
                        kubectl apply -f service.yaml -n ${KUBERNETES_NAMESPACE}
                        kubectl rollout status deployment/devsecops-flask-app -n ${KUBERNETES_NAMESPACE}
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
