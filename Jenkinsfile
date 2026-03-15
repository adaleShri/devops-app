pipeline {
    agent any

    environment {
        // Dynamic tagging for traceability (Industry Standard)
        DOCKER_IMAGE = "adaleshri/devopsexamapp"
        SONAR_SERVER = 'SonarQube'
        SCANNER_HOME = tool 'SonarScanner' // Tool name from Global Tool Config
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs() // Start with a fresh slate
            }
        }

        stage('Git Checkout') {
            steps {
                git url: 'https://github.com/adaleshri/devops-app.git', branch: 'main'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv("${SONAR_SERVER}") {
                        sh "${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=exam-app"
                    }
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

        stage('FileSystem Scan (Trivy)') {
            steps {
                // Scan source code for hardcoded secrets or vulnerabilities before building
                sh 'trivy fs --severity HIGH,CRITICAL --format table .'
            }
        }

        stage('Build & Tag Image') {
            steps {
                dir('backend') {
                    script {
                        dockerImage = docker.build("${DOCKER_IMAGE}")
                    }
                }
            }
        }

        stage('Image Scan (Trivy)') {
            steps {
                // Scan the actual Docker image for OS-level vulnerabilities
                sh "trivy image --severity HIGH,CRITICAL ${DOCKER_IMAGE}"
            }
        }

        stage('Push to Registry') {
            steps {
                script {
                    docker.withRegistry('', 'docker-creds') {
                        dockerImage.push()
                        dockerImage.push("latest") // Also update latest tag
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
            echo "❌ Pipeline failed at Build ${BUILD_NUMBER}. Investigating logs..."
            // In real envs, you'd add a Slack/Email notification here
        }
        always {
            // Clean up old images to prevent Jenkins disk from filling up
            sh 'docker image prune -f'
        }
    }
}
