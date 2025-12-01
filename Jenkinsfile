pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = "docker.io"
        IMAGE_NAME = "vothangtri/simple-nginx"
        GIT_CREDENTIAL = "git-cred"
        DOCKER_CREDENTIAL = "docker-cred"
    }

    stages {

        stage('Checkout') {
            steps {
                git credentialsId: "${GIT_CREDENTIAL}", url: "https://github.com/vottri/nginx-cicd-lab.git"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    IMAGE_TAG = "build-${env.BUILD_NUMBER}"
                    sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIAL}", usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh """
                    echo "$PASS" | docker login -u "$USER" --password-stdin ${DOCKER_REGISTRY}
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Update Helm values.yaml') {
            steps {
                sh """
                sed -i 's/tag:.*/tag: "${IMAGE_TAG}"/' charts/nginx/values.yaml
                """
            }
        }

        stage('Commit & Push Back') {
            steps {
                sh """
                  git config user.email "thangtrivo1991@gmail.com"
                  git config user.name "vottri"
                  git add charts/nginx/values.yaml
                  git commit -m "Update image tag to ${IMAGE_TAG}" || true
                  git push origin main
                """
            }
        }
    }
}
