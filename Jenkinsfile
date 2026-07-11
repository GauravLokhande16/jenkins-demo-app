pipeline {
    agent any

    // This defines the input field inside the Jenkins UI
    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'dev', description: 'Which branch do you want to test and deploy?')
    }

    environment {
        // Make sure Jenkins can find docker/kubectl regardless of Mac chip types
        PATH = "/usr/local/bin:/opt/homebrew/bin:${env.PATH}"
        IMAGE_NAME = "jenkins-demo-app"
        IMAGE_TAG  = "local"
    }

    stages {
        stage('Checkout') {
            steps {
                // This clean syntax handles pulling whatever branch you type into the UI parameter
                git branch: "${params.BRANCH_NAME}", 
                    url: 'https://github.com/GauravLokhande16/jenkins-demo-app.git'
                //    credentialsId: 'YOUR_GITHUB_CREDENTIALS_ID'  Remove this line if it's a public repository
            }
        }

        stage('Install Dependencies') {
            steps {
                // 'npm ci' is perfect for CI/CD as it installs exact versions from package-lock.json
                sh 'npm ci'
            }
        }

        stage('Code Quality Checks') {
            parallel {
                stage('Lint') {
                    steps {
                        echo 'Checking code quality with ESLint...'
                        sh 'npm run lint'
                    }
                }
                stage('Format Check') {
                    steps {
                        echo 'Checking code formatting with Prettier...'
                        sh 'npm run format:check'
                    }
                }
                stage('Test') {
                    steps {
                        echo 'Running unit tests...'
                        sh 'npm test'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            when {
                // This matches 'main' OR 'origin/main'
                expression { return env.BRANCH_NAME == params.BRANCH_NAME || params.BRANCH_NAME != '' }
            }
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                // This matches 'main' OR 'origin/main'
              expression { return env.BRANCH_NAME == params.BRANCH_NAME || params.BRANCH_NAME != '' }
            }
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