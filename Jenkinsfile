pipeline {
    agent any
    environment {
        SAFE_BRANCH = "${env.BRANCH_NAME.replaceAll('/', '-')}"
        IMAGE = "docker.io/jithu145/java-microservice:${SAFE_BRANCH}"
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
        stage('Docker Build & Push') {
            when {
                anyOf {
                    branch 'develop'
                    branch pattern: "release/.*", comparator: "REGEXP"
                    branch 'main'
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'fb16b1ba-d2e9-41bb-8654-d00d3b5b61e6', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                        docker build -t "$IMAGE" .
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                        docker push "$IMAGE"
                    '''
                }
            }
        }
        stage('Deploy to Staging') {
            when {
                branch pattern: "release/.*", comparator: "REGEXP"
            }
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONF')]) {
                    sh '''
                        sed -i "s|IMAGE_PLACEHOLDER|$IMAGE|" k8s/deployment.yaml
                        export KUBECONFIG=$KUBECONF
                        kubectl apply -f k8
