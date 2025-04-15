pipeline {
    agent any

    environment {
        IMAGE_NAME = "10.67.60.223:30002/yotta-app/hello-world-python"
        IMAGE_TAG = "latest"
        HARBOR_CREDENTIALS_ID = "harbor-credentials"
        GITHUB_CREDENTIALS_ID = "github-credentials"
        ARGOCD_SERVER = "10.67.60.223:32007"
        ARGOCD_APP_NAME = "yotta-app"
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    sh 'rm -rf argocd'

                    withCredentials([usernamePassword(credentialsId: GITHUB_CREDENTIALS_ID, usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                        sh '''
                            git clone https://$GIT_USER:$GIT_PASS@github.com/geetha-17/argocd.git
                        '''
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                        docker build -t $IMAGE_NAME:$IMAGE_TAG .
                    '''
                }
            }
        }

        stage('Login to Harbor') {
            steps {
                withCredentials([usernamePassword(credentialsId: HARBOR_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login 10.67.60.223:30002 -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh '''
                        docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Trigger ArgoCD Sync') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'argocd-auth-token', variable: 'ARGOCD_AUTH_TOKEN')]) {
                        sh '''
                            curl -k -X POST https://$ARGOCD_SERVER/api/v1/applications/$ARGOCD_APP_NAME/sync \
                            -H "Authorization: Bearer $ARGOCD_AUTH_TOKEN"
                        '''
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
