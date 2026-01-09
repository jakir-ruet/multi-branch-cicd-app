pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = "jakirbd/multi-branch-cicd-app"
        GIT_USER   = "jakir-ruet"
        GIT_EMAIL  = "jakir.ruet.bd@gmail.com"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Push Image') {
            when { branch 'master' } // adjust if your main branch is 'main'
            steps {
                script {
                    // Define image tag locally inside the script block
                    def IMAGE_TAG = "build-${BUILD_NUMBER}"

                    // Docker credentials
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        echo "Building Docker image ${IMAGE_NAME}:${IMAGE_TAG}"

                        sh """
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker logout
                        """
                    }
                }
            }
        }

        stage('Update K8s Manifest') {
            when { branch 'master' }
            steps {
                script {
                    // GitHub credentials for pushing manifest
                    withCredentials([usernamePassword(
                        credentialsId: 'github_creds',
                        usernameVariable: 'GIT_USERNAME',
                        passwordVariable: 'GIT_TOKEN'
                    )]) {
                        sh """
                        set -e
                        git config user.name "$GIT_USER"
                        git config user.email "$GIT_EMAIL"

                        git fetch origin
                        git checkout master
                        git reset --hard origin/master

                        sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" k8s/deployment.yml

                        git add k8s/deployment.yml
                        git diff --cached --quiet || git commit -m "Updated image to ${IMAGE_TAG}"
                        git push https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/jakir-ruet/multi-branch-cicd-app.git master
                        """
                    }
                }
            }
        }
    }
}
