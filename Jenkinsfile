pipeline {
    agent any
    environment {
        IMAGE = "your-dockerhub-username/java-microservice:${env.BRANCH_NAME}"
        SONARQUBE = 'SonarQube-Server'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('SonarQube Analysis') {
            when {
                anyOf {
                    branch pattern: "feature/.*", comparator: "REGEXP"
                    branch "develop"
                    branch pattern: "release/.*", comparator: "REGEXP"
                    branch "main"
                }
            }
            steps {
                withSonarQubeEnv("${SONARQUBE}") {
                    sh 'sonar-scanner'
                }
            }
        }
        stage('Docker Build & Push') {
            when {
                anyOf {
                    branch "develop"
                    branch pattern: "release/.*", comparator: "REGEXP"
                    branch "main"
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh """
                        docker build -t ${IMAGE} .
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                        docker push ${IMAGE}
                    """
                }
            }
        }
        stage('Deploy to Staging') {
            when {
                branch pattern: "release/.*", comparator: "REGEXP"
            }
            steps {
                sh """
                    sed -i 's|IMAGE_PLACEHOLDER|${IMAGE}|' k8s/deployment.yaml
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                """
            }
        }
        stage('Approval & Deploy to Prod') {
            when {
                branch 'main'
            }
            steps {
                input message: "Deploy to Production?"
                sh """
                    sed -i 's|IMAGE_PLACEHOLDER|${IMAGE}|' k8s/deployment.yaml
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                """
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
