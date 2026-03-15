pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "adaleshri/devopsexamapp"
        IMAGE_TAG = "${BUILD_NUMBER}"
        SONAR_SERVER = "SonarQube"
        SCANNER_HOME = tool 'Sonar-Scanner'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Git Checkout') {
            steps {
                git url: 'https://github.com/adaleshri/devops-app.git', branch: 'main'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONAR_SERVER}") {
                    sh """
                    ${SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectKey=exam-app \
                    -Dsonar.sources=.
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('File System Scan (Trivy)') {
            steps {
                sh 'trivy fs --severity HIGH,CRITICAL --format table . || true'
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('backend') {
                    script {
                        dockerImage = docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}")
                    }
                }
            }
        }

        stage('Image Scan (Trivy)') {
            steps {
                sh "trivy image --severity HIGH,CRITICAL ${DOCKER_IMAGE}:${IMAGE_TAG} || true"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('', 'docker-creds') {
                        dockerImage.push("${IMAGE_TAG}")
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                sh '''
                docker compose down --remove-orphans || true
                docker compose up -d
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Build ${BUILD_NUMBER} deployed successfully."
        }

        failure {
            echo "❌ Pipeline failed at build ${BUILD_NUMBER}"
            sh 'docker compose logs --tail=50 || true'
        }

        always {
            sh 'docker image prune -f'
        }
    }
}
