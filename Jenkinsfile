pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "adaleshri/devopsexamapp:latest"
        EKS_CLUSTER = "devops-app"
        K8S_NAMESPACE = "exam-app"
        AWS_REGION = "ap-south-1"
    }

    stages {
         stage('Git Checkout') {
            steps {
                git url: 'https://github.com/adaleshri/devops-app.git',
                    branch: 'main'
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

        stage('Push to Docker Hub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-creds', toolName: 'docker') {
                        sh "docker push ${DOCKER_IMAGE}"
                    }
                }
            }
        }

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

    }
}
