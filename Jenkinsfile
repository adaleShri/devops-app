pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "adaleshri/devopsexamapp"
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
                    sh "docker build -t ${DOCKER_IMAGE}:latest ."
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry(credentialsId: 'docker-creds') {
                    sh '''
                    docker tag ${DOCKER_IMAGE}:latest ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    docker push ${DOCKER_IMAGE}:latest
                    '''
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

                    sh '''
                    aws eks update-kubeconfig --name devops-app --region ap-south-1

                    kubectl create namespace exam-app --dry-run=client -o yaml | kubectl apply -f -

                    kubectl apply -f deployment.yml -n exam-app
                    kubectl apply -f service.yml -n exam-app

                    kubectl rollout status deployment/exam-app -n exam-app
                    '''
                }
            }
        }
    }
}
