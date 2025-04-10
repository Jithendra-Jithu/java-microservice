pipeline {
    agent any
    environment {
        IMAGE = "docker.io/jithu145/java-microservice:${env.BRANCH_NAME.replaceAll('/', '-')}"
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
                    branch "develop"
                    branch pattern: "release/.*", comparator: "REGEXP"
                    branch "main"
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'fb16b1ba-d2e9-41bb-8654-d00d3b5b61e6', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                        docker build -t $IMAGE .
                        docker push $IMAGE
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
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                    '''
                }
            }
        }

        stage('Approval & Deploy to Prod') {
            when {
                branch 'main'
            }
            steps {
                input message: "Deploy to Production?"
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONF')]) {
                    sh '''
                        sed -i "s|IMAGE_PLACEHOLDER|$IMAGE|" k8s/deployment.yaml
                        export KUBECONFIG=$KUBECONF
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                    '''
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
