pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = "jakirbd/multi-branch-cicd-app"
        GIT_USER   = "jakir-ruet"
        GIT_EMAIL  = "jakir.ruet.bd@gmail.com"
        // IMAGE_TAG will be set dynamically in the script block
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Push Docker Image') {
            when { branch 'master' } // Change if your main branch is 'main'
            steps {
                script {
                    // Set IMAGE_TAG in env so it is available in other stages
                    env.IMAGE_TAG = "build-${BUILD_NUMBER}"

                    echo "Building Docker image ${IMAGE_NAME}:${IMAGE_TAG}"

                    // DockerHub credentials
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                            # Build Docker image
                            docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .

                            # Login to DockerHub
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                            # Push image
                            docker push ${IMAGE_NAME}:${IMAGE_TAG}

                            # Logout
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

                            # Update the Docker image in your K8s deployment YAML
                            sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" k8s/deployment.yml

                            git add k8s/deployment.yml
                            git diff --cached --quiet || git commit -m "Updated image to ${IMAGE_TAG}"
                            git push https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/jakir-ruet/multi-branch-cicd-app.git master
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when { branch 'master' }
            steps {
                script {
                    // Assumes kubectl is configured on Jenkins agent
                    sh """
                        kubectl apply -f k8s/deployment.yml
                        kubectl rollout status deployment/your-deployment-name
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully: ${IMAGE_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
