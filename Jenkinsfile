pipeline {
    agent any
    
    tools {
        maven 'maven3'  // Make sure this name matches your Jenkins tool configuration
        jdk 'JDK8'      // Make sure this name matches your Jenkins tool configuration
    }
    
    environment {
        AWS_REGION = 'us-east-1'
        ECR_REGISTRY = '935598635277.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REPOSITORY = 'new/imp-repo'
        IMAGE_TAG = "${BUILD_NUMBER}"
        AWS_CREDENTIALS_ID = 'aws-ecr-credentials'
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
         
        stage('Build Docker Image') {
            steps {
                script {
                    // Build image with ECR-compatible naming
                    dockerImage = docker.build("${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}", "./docker")
                }
            }
        }
        
        stage('Push to ECR') {
            steps {
                script {
                    // Method 1: Using ECR Plugin (Recommended)
                    docker.withRegistry("https://${ECR_REGISTRY}", "ecr:${AWS_REGION}:${AWS_CREDENTIALS_ID}") {
                        dockerImage.push("${IMAGE_TAG}")
                        dockerImage.push("latest")
                    }
                    
                    // Alternative Method 2: Using AWS CLI (if Method 1 fails)
                    /*
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', 
                                    credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
                        sh """
                            aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                            docker push ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                            docker push ${ECR_REGISTRY}/${ECR_REPOSITORY}:latest
                        """
                    }
                    */
                }
            }
        }
        
        stage('Clean Up') {
            steps {
                sh "docker rmi ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG} || true"
                sh "docker rmi ${ECR_REGISTRY}/${ECR_REPOSITORY}:latest || true"
            }
        }
    }
    
    post {
        success {
            echo "Pipeline completed successfully! Image pushed to ECR: ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}"
        }
        failure {
            echo "Pipeline failed! Check logs for details."
        }
    }
}
