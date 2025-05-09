pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = 'devsecops-flask-app'
        DOCKER_HUB_REPO = 'soumil22/devsecops-flask-app'
        KUBERNETES_NAMESPACE = 'default'
        SONARQUBE_ENV = 'MySonarQube'  // Must match Jenkins SonarQube configuration name
    }

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/Soumil2002/Devsecops-cicd-pipeline.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'sonar-scanner \
                        -Dsonar.projectKey=devsecops-flask-app \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.login=$SONAR_AUTH_TOKEN'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE_NAME .'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy image --exit-code 0 --severity HIGH,CRITICAL $DOCKER_IMAGE_NAME'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker tag $DOCKER_IMAGE_NAME:latest $DOCKER_HUB_REPO:latest'
                    sh 'docker push $DOCKER_HUB_REPO:latest'
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml --namespace=$KUBERNETES_NAMESPACE'
                sh 'kubectl apply -f k8s/service.yaml --namespace=$KUBERNETES_NAMESPACE'
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
