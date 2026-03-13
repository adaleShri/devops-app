pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "adaleshri/devopsexamapp:latest"
        EKS_CLUSTER = "devops-app"
        K8S_NAMESPACE = "exam-app"
        AWS_REGION = "ap-south-1"  // Update to your region
    }

    stages {
        stage('Git Checkout') {
            steps {
                git url: 'https://github.com/adaleshri/devops-app.git', 
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
// NEW STAGE: Push to Docker Hub
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
         stages {
        // Existing stages (Git Checkout, Build, Push) remain the same
        
        stage('Deploy to EKS') {
            steps {
                script {
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'aws-creds',
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                    ]]) {
                        sh """
                        # Configure EKS access
                        aws eks update-kubeconfig --name ${EKS_CLUSTER} --region ${AWS_REGION}
                        
                        # Create namespace if not exists
                        kubectl create namespace ${K8S_NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -
                        
                        # Create image pull secret
                        kubectl create secret docker-registry dockerhub-creds \\
                            --docker-server=https://index.docker.io/v1/ \\
                            --docker-username=kastrov \\
                            --docker-password=\$(cat /var/jenkins_home/docker-creds/password) \\
                            --namespace=${K8S_NAMESPACE} \\
                            --dry-run=client -o yaml | kubectl apply -f -
                        
                        # Apply Kubernetes manifests from root
                        kubectl apply -f deployment.yml
                        kubectl apply -f service.yml
                        
                        # Verify deployment
                        kubectl rollout status deployment/exam-app -n ${K8S_NAMESPACE}
                        """
                    }
                }
            }
        }
    }
}
