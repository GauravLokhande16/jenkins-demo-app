pipeline {
    agent any

    environment {
        // Make sure Jenkins can find docker/kubectl regardless of Mac chip type
        PATH = "/usr/local/bin:/opt/homebrew/bin:${env.PATH}"
        IMAGE_NAME = "jenkins-demo-app"
        IMAGE_TAG  = "local"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install & Test') {
            steps {
                sh 'npm ci'
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    kubectl rollout status deployment/jenkins-demo-app --timeout=120s
                '''
            }
        }
    }

    post {
        success { echo "Deployed successfully. Try: curl localhost" }
        failure { echo "Something failed — check the stage logs above." }
    }
}