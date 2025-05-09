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
        // Stage 0: Verify SonarQube Connection
        stage('Pre-Check: SonarQube Availability') {
            steps {
                script {
                    try {
                        sh """
                            curl -sI http://54.225.5.230:9000 | head -1 | grep 200
                            echo "SonarQube server is reachable"
                        """
                    } catch (Exception e) {
                        error "SonarQube server is not reachable at http://54.225.5.230:9000"
                    }
                }
            }
        }

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
                script {
                    withSonarQubeEnv('MySonarQube') {
                        sh """
                            ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                            -Dsonar.projectKey=DevSecOps-Flask-App \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=\${SONAR_HOST_URL} \
                            -Dsonar.login=\${SONAR_AUTH_TOKEN} \
                            -Dsonar.connectionTimeout=60000 \
                            -Dsonar.socketTimeout=60000
                        """
                    }
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
            script {
                // Send notification based on build status
                if (currentBuild.result == 'SUCCESS') {
                    slackSend(color: 'good', message: "Pipeline SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
                } else {
                    slackSend(color: 'danger', message: "Pipeline FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
                }
            }
        }
    }
}
