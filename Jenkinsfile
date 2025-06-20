pipeline {
    agent any

    tools {
        nodejs 'NodeJS 7.8.0'
    }

    environment {
        BRANCH = "${env.BRANCH_NAME}"
        PORT = BRANCH == 'main' ? '3000' : '3001'
        IMAGE_NAME = BRANCH == 'main' ? 'nodemain:v1.0' : 'nodedev:v1.0'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Lint Dockerfile') {
            steps {
                sh 'hadolint Dockerfile'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Scan Image') {
            steps {
                sh "trivy image ${IMAGE_NAME} || true"
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def container = BRANCH == 'main' ? 'main_container' : 'dev_container'
                    sh """
                        docker ps -q --filter "name=${container}" | xargs -r docker stop
                        docker ps -a -q --filter "name=${container}" | xargs -r docker rm
                        docker run -d --name ${container} -p ${PORT}:3000 ${IMAGE_NAME}
                    """
                }
            }
        }

        stage('Push Image (Optional)') {
            when {
                anyOf {
                    branch 'main'
                    branch 'dev'
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker tag ${IMAGE_NAME} $DOCKER_USER/${IMAGE_NAME}
                        docker push $DOCKER_USER/${IMAGE_NAME}
                    """
                }
            }
        }
    }
}
