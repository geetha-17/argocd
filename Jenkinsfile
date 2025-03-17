pipeline {
    agent any

    environment {
        IMAGE_NAME = 'geetha8500/hello-world-python'
        DOCKER_CREDENTIALS_ID = '41380629-0631-44da-ba4d-add2b73c5926'
        GIT_REPO = 'https://github.com/geetha-17/argocd.git'
        GIT_BRANCH = 'main'
        ARGOCD_SERVER = 'http://localhost:8080'
        ARGOCD_AUTH_TOKEN = credentials('argocd-auth-token')  // Store ArgoCD API token in Jenkins credentials
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    // Remove any previous copies of the repository to ensure a fresh clone
                    sh 'rm -rf hello-world-python || true'
                    sh 'git clone https://github.com/geetha-17/argocd.git'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image with the specified tag
                    sh 'docker build -t $IMAGE_NAME:latest hello-world-python'
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: '41380629-0631-44da-ba4d-add2b73c5926', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        // Login to Docker Hub using the credentials stored in Jenkins
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Push the Docker image to the Docker Hub
                    sh 'docker push $IMAGE_NAME:latest'
                }
            }
        }

        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    // Change to the ArgoCD repository directory
                    dir('argocd/manifests') {
                        // Update the image reference in the deployment.yaml file
                        sh 'sed -i "s|image: .*|image: $IMAGE_NAME:latest|" deployment.yaml'
                        // Configure Git to use Jenkins as the user
                        sh 'git config --global user.email "jenkins@localhost"'
                        sh 'git config --global user.name "Jenkins"'
                        // Commit and push the changes to the repository
                        sh 'git add deployment.yaml'
                        sh 'git commit -m "Updated image to $IMAGE_NAME:latest"'
                        sh 'git push origin $GIT_BRANCH'
                    }
                }
            }
        }

        stage('Trigger ArgoCD Sync') {
            steps {
                script {
                    // Trigger the sync for the ArgoCD application to deploy the updated image
                    sh '''
                    curl -X POST "$ARGOCD_SERVER/api/v1/applications/hello-world-python/sync" \
                    -H "Authorization: Bearer $ARGOCD_AUTH_TOKEN"
                    '''
                }
            }
        }
    }

    post {
        failure {
            echo 'Build or Deployment Failed!'
        }
        success {
            echo 'Build, Push, and Deployment Successful!'
        }
    }
}
