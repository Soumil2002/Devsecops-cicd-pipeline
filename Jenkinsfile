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
