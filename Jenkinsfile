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
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        IMAGE_NAME = "${ECR_REGISTRY}/${ECR_REPOSITORY}"
        AWS_CREDENTIALS_ID = 'aws-credentials'  // Update this to your actual Jenkins credential ID
    }
    stages {
        stage('Build Maven') {
            steps {
                sh 'mvn clean install package'
            }
        }
        stage('Copy Artifact') {
            steps {
                sh 'cp -r target/*.jar docker'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}", "./docker")
                }
            }
        }
        stage('Push to ECR') {
            steps {
                script {
                    docker.withRegistry("https://${ECR_REGISTRY}", "ecr:${AWS_REGION}:${AWS_CREDENTIALS_ID}") {
                        dockerImage.push("${IMAGE_TAG}")
                        dockerImage.push("latest")
                    }
                }
            }
        }
    }
}
