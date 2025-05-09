pipeline {
    agent any

    environment {
        // Tool and Path
        SONAR_SCANNER_HOME = tool name: 'sonar-scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
        PATH = "${env.SONAR_SCANNER_HOME}/bin:${env.PATH}"
        
        // Docker and K8s Config
        DOCKER_IMAGE_NAME = 'devsecops-flask-app'
        DOCKER_HUB_REPO = 'soumil22/devsecops-flask-app'
        KUBERNETES_CLUSTER_NAME = 'devsecops-cluster'
        KUBERNETES_NAMESPACE = 'default'
    }

    stages {
        stage('Pre-Check: SonarQube Availability') {
            steps {
                script {
                    def response = sh(script: "curl -sI http://54.225.5.230:9000 | grep '200 OK' || true", returnStatus: true)
                    if (response != 0) {
                        error "SonarQube server is not reachable at http://54.225.5.230:9000"
                    } else {
                        echo "SonarQube server is reachable"
                    }
                }
            }
        }

        stage('Checkout Code') {
            steps {
                checkout([$class: 'GitSCM', 
                          branches: [[name: '*/main']],
                          extensions: [], 
                          userRemoteConfigs: [[url: 'https://github.com/Soumil2002/Devsecops-cicd-pipeline.git']]
                ])
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonarQube') {
                    withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_AUTH_TOKEN')]) {
                        sh """
                            sonar-scanner \
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

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE_NAME} ."
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    try {
                        sh "trivy image --exit-code 1 --severity HIGH,CRITICAL ${DOCKER_IMAGE_NAME}"
                        echo "No critical vulnerabilities found"
                    } catch (Exception e) {
                        error "Trivy scan failed due to critical vulnerabilities"
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials', 
                    usernameVariable: 'DOCKER_USER', 
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        docker login -u $DOCKER_USER -p $DOCKER_PASS
                        docker tag ${DOCKER_IMAGE_NAME}:latest ${DOCKER_HUB_REPO}:latest
                        docker push ${DOCKER_HUB_REPO}:latest
                        docker logout
                    """
                }
            }
        }

        stage('Manual Approval') {
            steps {
                input message: 'Approve deployment to EKS?'
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    sh """
                        aws eks --region us-east-1 update-kubeconfig --name ${KUBERNETES_CLUSTER_NAME}
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
                def status = currentBuild.result ?: 'SUCCESS'
                def color = status == 'SUCCESS' ? 'good' : 'danger'
                slackSend(color: color, message: "Pipeline ${status}: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
            }
        }
    }
}
