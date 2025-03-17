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
                    sh 'rm -rf hello-world-python || true'
                    sh 'git clone https://geetha-17:${GIT_PASSWORD}@github.com/geetha-17/argocd.git'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $IMAGE_NAME:latest hello-world-python'
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: '41380629-0631-44da-ba4d-add2b73c5926', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh 'docker push $IMAGE_NAME:latest'
                }
            }
        }

        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    // Clone the ArgoCD repo
                    sh 'rm -rf argocd || true'
                    sh 'git clone $GIT_REPO'
                    
                    // Update image in deployment.yaml
                    sh '''
                    cd argocd/manifests
                    sed -i "s|image: .*|image: $IMAGE_NAME:latest|" deployment.yaml
                    git config --global user.email "jenkins@localhost"
                    git config --global user.name "Jenkins"
                    git add deployment.yaml
                    git commit -m "Updated image to $IMAGE_NAME:latest"
                    git push origin $GIT_BRANCH
                    '''
                }
            }
        }

        stage('Trigger ArgoCD Sync') {
            steps {
                script {
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
