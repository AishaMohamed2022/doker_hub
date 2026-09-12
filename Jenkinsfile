pipeline {

    agent any

    environment {
        DOCKER_IMAGE = "3isha/task-jenkins"
        DOCKER_CREDENTIALS = "docker"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Cloning GitHub repository...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'

                sh '''
                    docker build -t ${DOCKER_IMAGE}:build-${BUILD_NUMBER} .
                '''
            }
        }

        stage('Docker Login') {
            steps {
                echo 'Logging in to Docker Hub...'

                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIALS}",
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Pushing image to Docker Hub...'

                sh '''
                    docker push ${DOCKER_IMAGE}:build-${BUILD_NUMBER}
                '''
            }
        }
    }

    post {

        success {
            echo """
========================================
PIPELINE SUCCESSFUL
========================================

Docker Image:
${DOCKER_IMAGE}:build-${BUILD_NUMBER}

Build Number:
${BUILD_NUMBER}

Docker image was successfully pushed to Docker Hub.

========================================
"""
        }

        failure {
            echo """
========================================
PIPELINE FAILED
========================================

Project:
${JOB_NAME}

Build Number:
${BUILD_NUMBER}

Check Jenkins Console Output for the error.

========================================
"""
        }

        always {
            sh '''
                docker logout || true
            '''
        }
    }
}
