pipeline {
    agent any
    tools {
        maven 'maven3'
        jdk 'JDK8'
    }
    environment {
        AWS_REGION = 'us-east-1'
        ECR_REGISTRY = '935598635277.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REPOSITORY = 'new/imp-repo'
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${ECR_REGISTRY}/${ECR_REPOSITORY}"
        AWS_CREDENTIALS_ID = 'aws-ecr-credentials' // Set this to match your Jenkins credentials ID
    }
    stages {      
        stage('Build Maven') {
            steps { 
                sh 'pwd'
                sh 'mvn clean install package'
            }
        }
        stage('Copy Artifact') {
            steps { 
                sh 'pwd'
                sh 'cp -r target/*.jar docker'
            }
        }
        stage('Build Docker image') {
            steps {
                script {
                    // Build image with ECR-compatible name (not DockerHub!)
                    dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}", "./docker")
                }
            }
        }
        stage('Push to ECR') {
            steps {
                script {
                    // Authenticate and push using Jenkins ECR credentials (requires Docker Pipeline + AWS/ECR plugins)
                    docker.withRegistry("https://${ECR_REGISTRY}", "ecr:${AWS_REGION}:${AWS_CREDENTIALS_ID}") {
                        dockerImage.push("${IMAGE_TAG}")         // Push with build number tag
                        dockerImage.push("latest")               // Optionally push as latest as well
                    }
                }
            }
        }
    }
    post {
        success {
            echo "Image pushed to ECR as ${IMAGE_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}
