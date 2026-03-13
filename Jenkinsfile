pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "adaleshri/devopsexamapp:latest"
        EKS_CLUSTER = "devops-app"
        K8S_NAMESPACE = "exam-app"
        AWS_REGION = "ap-south-1"  // Update to your region
    }
    }

    stages {
        stage('Git Checkout') {
            steps {
                git url: 'https://github.com/adaleshri/devops-exam-app.git', 
                    branch: 'main'
            }
        }

        stage('Verify Docker Compose') {
            steps {
                sh '''
                docker compose version || { echo "Docker Compose not available"; exit 1; }
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('backend') {
                    script {
                        withDockerRegistry(credentialsId: 'docker-creds', toolName: 'docker') {
                            sh "docker build -t ${DOCKER_IMAGE} ."
                        }
                    }
                }
            }
        }
        / NEW STAGE: Push to Docker Hub
        stage('Push to Docker Hub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-creds', toolName: 'docker') {
                        sh """
                        docker tag ${DOCKER_IMAGE} ${DOCKER_IMAGE}
                        docker push ${DOCKER_IMAGE}
                        """
                    }
                }
            }
        }
        pipeline {
agent any

```
environment {
    DOCKER_IMAGE = "adaleshri/devopsexamapp"
    SCANNER_HOME = tool 'sonar-scanner'
    EKS_CLUSTER = "devops-app"
    K8S_NAMESPACE = "exam-app"
    AWS_REGION = "ap-south-1"
}

stages {

    stage('Git Checkout') {
        steps {
            git url: 'https://github.com/adaleShri/devops-app.git', branch: 'main'
        }
    }

    stage('File System Scan') {
        steps {
            sh 'trivy fs --scanners vuln,misconfig --format table -o trivy-fs-report.html .'
        }
    }

    stage('SonarQube Analysis') {
        steps {
            withSonarQubeEnv('sonar-server') {
                sh """
                ${SCANNER_HOME}/bin/sonar-scanner \
                -Dsonar.projectName=devops-exam-app \
                -Dsonar.projectKey=devops-exam-app \
                -Dsonar.sources=. \
                -Dsonar.exclusions=**/*.java \
                -Dsonar.host.url=http://localhost:9000
                """
            }
        }
    }

    stage('Verify Docker') {
        steps {
            sh 'docker --version'
        }
    }

    stage('Build Docker Image') {
        steps {
            dir('backend') {
                sh "docker build -t ${DOCKER_IMAGE}:latest ."
            }
        }
    }

    stage('Push to Docker Hub') {
        steps {
            withDockerRegistry(credentialsId: 'docker-creds') {
                sh """
                docker tag ${DOCKER_IMAGE}:latest ${DOCKER_IMAGE}:${BUILD_NUMBER}
                docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                docker push ${DOCKER_IMAGE}:latest
                """
            }
        }
    }

    stage('Deploy to EKS') {
        steps {
            withCredentials([[
                $class: 'AmazonWebServicesCredentialsBinding',
                credentialsId: 'aws-creds',
                accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
            ]]) {

                sh """
                aws eks update-kubeconfig --name ${EKS_CLUSTER} --region ${AWS_REGION}

                kubectl create namespace ${K8S_NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -

                kubectl apply -f deployment.yml -n ${K8S_NAMESPACE}
                kubectl apply -f service.yml -n ${K8S_NAMESPACE}

                kubectl rollout status deployment/exam-app -n ${K8S_NAMESPACE}
                """
            }
        }
    }
}

post {
    success {
        echo "Pipeline completed successfully. Application deployed to EKS."
    }
    failure {
        echo "Pipeline failed. Check logs."
    }
}
```

}
