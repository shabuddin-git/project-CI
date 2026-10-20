
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'shabdoc/streamlit-app:${GIT_COMMIT}'
        EKS_CLUSTER_NAME = "shab-eks-cluster"
        AWS_REGION = "us-east-2"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/shabuddin-git/project-CI.git'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t "$IMAGE_NAME" .
                '''
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-hub-cred',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    docker push "$IMAGE_NAME"
                '''
            }
        }

        stage('update-kubeconfig') {
            steps {
                sh '''
                    aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}
                '''
            }
        }

        stage('deploy to kubernetes') {
            steps {
                withKubeConfig(
                    caCertificate: '',
                    clusterName: 'shab-eks-cluster',
                    contextName: '',
                    credentialsId: 'kube',
                    namespace: 'default',
                    restrictKubeConfigAccess: false,
                    serverUrl: 'https://FA55DA12F65E1ACAAE69CDB7C5C19AAB.gr7.us-east-2.eks.amazonaws.com'
                ) {
                    sh '''
                        sed -i "s|replace|${IMAGE_NAME}|g" Deployment.yaml
                        kubectl apply -f Deployment.yaml
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }
    }
}
