pipeline {
    agent any

    environment {
        APP_NAME    = 'foodexpress-api'
        IMAGE_NAME  = 'foodexpress-api'
        APP_PORT    = '5000'
        REPO_URL    = 'https://github.com/Nikola-Limpet/Devop-class-assignment6'
    }

    tools {
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

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    script {
                        def scannerHome = tool 'SonarScanner'
                        sh """
                            ${scannerHome}/bin/sonar-scanner \\
                              -Dsonar.projectKey=foodexpress-api \\
                              -Dsonar.projectName="FoodExpress API" \\
                              -Dsonar.sources=. \\
                              -Dsonar.exclusions=node_modules/**
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Trivy Code Scan') {
            steps {
                sh 'trivy fs --exit-code 0 --severity HIGH,CRITICAL --format table .'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
                sh "docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest"
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --exit-code 0 --severity HIGH,CRITICAL --format table ${IMAGE_NAME}:latest"
            }
        }

        stage('Deploy Container') {
            steps {
                sh "docker rm -f ${APP_NAME} || true"
                sh "docker run -d --name ${APP_NAME} -p ${APP_PORT}:5000 ${IMAGE_NAME}:latest"
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'sleep 5'
                sh "docker ps | grep ${APP_NAME}"
                sh "curl -s http://localhost:${APP_PORT}/menu || true"
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
