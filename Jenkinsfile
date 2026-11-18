pipeline {
    agent any

    environment {
        IMAGE_NAME = 'midterm-api'
        CONTAINER_NAME = 'midterm-api'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Train Model') {
            steps {
                sh '''
                    docker run --rm \
                      -v "$WORKSPACE:/app" \
                      -w /app \
                      -e GIT_COMMIT="$GIT_COMMIT" \
                      python:3.12-slim \
                      sh -c "pip install --no-cache-dir -r requirements.txt && python generate_dataset.py --config config.json --output dataset/train.csv && python train.py"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker rm -f ${CONTAINER_NAME} || true
                    docker run -d --restart unless-stopped \
                      --name ${CONTAINER_NAME} \
                      -p 8000:8000 \
                      ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Test API') {
            steps {
                sh '''
                    sleep 5
                    curl --fail http://localhost:8000/health
                    curl --fail http://localhost:8000/metrics
                '''
            }
        }
    }
}
    post {
        always {
            emailext(
                to: 'qasimalik@gmail.com',
                subject: "FA23-BDS-019 - Jenkins Build #${BUILD_NUMBER} - ${currentBuild.currentResult}",
                body: """
Student ID: FA23-BDS-019
Build Number: ${BUILD_NUMBER}
Build Status: ${currentBuild.currentResult}
Build URL: ${BUILD_URL}
"""
            )
        }
    }
