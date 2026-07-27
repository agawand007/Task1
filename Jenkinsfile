pipeline {
    agent any
    
    environment {
        // Docker Hub credentials (configured in Jenkins)
        DOCKER_CREDENTIALS = credentials('docker-hub-credentials')
        DOCKER_REGISTRY = 'docker.io'
        IMAGE_NAME = 'argwims007/Task2'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    options {
        // Keep only the last 10 builds
        buildDiscarder(logRotator(numToKeepStr: '10'))
        // Add timestamps to console output
        timestamps()
        // Timeout after 30 minutes
        timeout(time: 30, unit: 'MINUTES')
    }
    
    stages {
        stage('Checkout') {
            steps {
                script {
                    echo '============================================'
                    echo 'Checking out code from Git repository...'
                    echo '============================================'
                }
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo '============================================'
                    echo 'Building Docker image...'
                    echo '============================================'
                    echo "Image: ${IMAGE_NAME}:${IMAGE_TAG}"
                    
                    // Build the Docker image
                    sh '''
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                    '''
                }
            }
        }
        
        stage('Login to Docker Hub') {
            steps {
                script {
                    echo '============================================'
                    echo 'Logging in to Docker Hub...'
                    echo '============================================'
                    
                    // Login using credentials bound from Jenkins
                    sh '''
                        echo $DOCKER_CREDENTIALS_PSW | docker login -u $DOCKER_CREDENTIALS_USR --password-stdin
                    '''
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    echo '============================================'
                    echo 'Pushing image to Docker Hub...'
                    echo '============================================'
                    
                    sh '''
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:latest
                    '''
                }
            }
        }
        
        stage('Verify Image') {
            steps {
                script {
                    echo '============================================'
                    echo 'Verifying pushed image...'
                    echo '============================================'
                    
                    sh '''
                        echo "Image pushed successfully!"
                        docker inspect ${IMAGE_NAME}:${IMAGE_TAG} | head -20
                    '''
                }
            }
        }
    }
    
    post {
        always {
            script {
                echo '============================================'
                echo 'Cleaning up Docker login...'
                echo '============================================'
                
                // Logout from Docker Hub for security
                sh '''
                    docker logout || true
                '''
            }
        }
        
        success {
            echo 'Pipeline completed successfully!'
            echo "Image ${IMAGE_NAME}:${IMAGE_TAG} published to Docker Hub"
        }
        
        failure {
            echo 'Pipeline failed! Check the logs above for details.'
        }
    }
}
