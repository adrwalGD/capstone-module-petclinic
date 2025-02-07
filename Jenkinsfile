pipeline {
    agent any

    environment {
        IMAGE_NAME = "spring-petclinic"
        REGISTRY = "ddd.azurecr.io"
    }

    tools {
        maven 'maven'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Static Code Analysis') {
            steps {
                echo "Running static code analysis..."
                sh 'mvn checkstyle:check'
                sh 'mvn spotbugs:check'
            }
        }

        stage('Run Tests') {
            steps {
                echo "Running unit tests..."
                sh 'mvn test'
            }
        }

        stage('Build Application') {
            steps {
                echo "Building the application..."
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Create Artifact') {
            steps {
                script {
                    def shortCommit = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    def imageTag = "${REGISTRY}/${IMAGE_NAME}:${shortCommit}"
                    env.IMAGE_TAG = imageTag
                    echo "Generated artifact tag: ${imageTag}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh "docker build -t ${IMAGE_TAG} ."
            }
        }

        stage('Push Artifact to ACR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'azure-acr-credentials', usernameVariable: 'ACR_USER', passwordVariable: 'ACR_PASSWORD')]) {
                    echo "Logging in to Azure Container Registry..."
                    sh "echo ${ACR_PASSWORD} | docker login ${REGISTRY} -u ${ACR_USER} --password-stdin"
                    echo "Pushing image to ACR..."
                    sh "docker push ${IMAGE_TAG}"
                }
            }
        }
    }

    post {
        success {
            echo "PR pipeline completed successfully."
        }
        failure {
            echo "PR pipeline failed."
        }
    }
}
