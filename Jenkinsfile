pipeline {
    agent any

    environment {
        IMAGE_NAME = "geetha-17/hello-world-python"
        IMAGE_TAG = "latest"
        DOCKER_CREDENTIALS_ID = "41380629-0631-44da-ba4d-add2b73c5926"
        GITHUB_CREDENTIALS_ID = "b213f9bd-9d94-4318-aec9-aabb24118bfd"
        ARGOCD_SERVER = "localhost:8080"
        ARGOCD_APP_NAME = "hello-world-python"
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    // Clean up the workspace to prevent conflicts
                    sh 'rm -rf argocd'
                    
                    withCredentials([usernamePassword(credentialsId: GITHUB_CREDENTIALS_ID, usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                        sh 'git clone https://$GIT_USER:$GIT_PASS@github.com/geetha-17/argocd.git'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $IMAGE_NAME:$IMAGE_TAG ."
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh "docker push $IMAGE_NAME:$IMAGE_TAG"
                }
            }
        }

        stage('Trigger ArgoCD Sync') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'argocd-auth-token', variable: 'ARGOCD_AUTH_TOKEN')]) {
                        sh """
                            curl -k -X POST $ARGOCD_SERVER/api/v1/applications/$ARGOCD_APP_NAME/sync \
                            -H "Authorization: Bearer $ARGOCD_AUTH_TOKEN"
                        """
                    }
                }
            }
        }
    }

    post {
        failure {
            echo 'Build or Deployment Failed!'
        }
    }
}
