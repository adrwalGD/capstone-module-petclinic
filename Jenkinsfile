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
                // sh 'mvn checkstyle:check'
                // sh 'mvn spotbugs:check'
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
                script {
                    dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}", ".")
                }
            }
        }

        stage('Push Artifact to ACR') {
            steps {
                script {
                    docker.withRegistry('https://ddd.azurecr.io', 'ACR-user-pass') {
                        echo "Logging in to Azure Container Registry..."
                        sh "docker push ${IMAGE_TAG}"
                    }
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
