pipeline {
    agent any

    environment {
        APP_NAME    = 'foodexpress-api'
        IMAGE_NAME  = 'foodexpress-api'
        APP_PORT    = '5000'
        REPO_URL    = 'https://github.com/Nikola-Limpet/Devop-class-assignment6'
    }

    tools {
        // Must match the name in Global Tool Configuration
        nodejs 'NodeJS'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                deleteDir()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: "${REPO_URL}"
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        // REMOVED THE STRAY BRACE THAT WAS HERE

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
                sh "docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest"
            }
        }

        stage('Deploy Container') {
            steps {
                // Stops and removes existing container if it exists
                sh "docker rm -f ${APP_NAME} || true"
                sh "docker run -d --name ${APP_NAME} -p ${APP_PORT}:5000 ${IMAGE_NAME}:latest"
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    sh 'sleep 5'
                    sh "docker ps | grep ${APP_NAME}"
                    // Using || true prevents the pipeline from failing if the endpoint isn't ready
                    sh "curl -s http://localhost:${APP_PORT}/menu || true"
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
