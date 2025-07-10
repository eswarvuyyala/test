pipeline {
    agent any

    environment {
        // GitHub repo
        GIT_REPO = 'https://github.com/eswarvuyyala/react-app.git'

        // SonarQube settings
        SONAR_HOST = 'http://13.201.203.112:9000'
        SONAR_PROJECT_KEY = 'react-app'
        SONAR_CREDENTIALS_ID = 'new-test-sonar1'

        // Docker image info
        IMAGE_NAME = 'react-app'
        IMAGE_TAG = "v1.0.${BUILD_NUMBER}"
        FULL_IMAGE_NAME = "${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: "${GIT_REPO}", branch: 'main'
            }
        }

        stage('SonarQube Scan') {
            environment {
                SONAR_TOKEN = credentials("${SONAR_CREDENTIALS_ID}")
            }
            steps {
                sh """
                    sonar-scanner \
                      -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                      -Dsonar.sources=. \
                      -Dsonar.host.url=${SONAR_HOST} \
                      -Dsonar.login=${SONAR_TOKEN}
                """
            }
        }

        stage('Remove Existing Docker Image (if any)') {
            steps {
                script {
                    sh "docker rmi -f ${FULL_IMAGE_NAME} || true"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${FULL_IMAGE_NAME} ."
            }
        }

        stage('Trivy Scan Docker Image') {
            steps {
                sh """
                    trivy image --exit-code 0 --severity HIGH,CRITICAL --format table --output trivy-report.txt ${FULL_IMAGE_NAME}
                    cat trivy-report.txt
                """
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully.'
        }
        failure {
            echo '❌ Pipeline failed.'
        }
    }
}
