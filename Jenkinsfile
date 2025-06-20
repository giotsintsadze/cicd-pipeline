pipeline {
    agent any

    tools {
        nodejs 'nodejs'
    }

    environment {
        BRANCH = "${env.BRANCH_NAME}"
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
                script {
                    def imageName = env.BRANCH == 'main' ? 'nodemain:v1.0' : 'nodedev:v1.0'
                    env.IMAGE_NAME = imageName
                    sh "docker build -t ${env.IMAGE_NAME} ."
                }
            }
        }

        stage('Scan Image') {
            steps {
                sh "trivy image ${env.IMAGE_NAME} || true"
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def container = env.BRANCH == 'main' ? 'main_container' : 'dev_container'
                    def port = env.BRANCH == 'main' ? '3000' : '3001'
                    sh """
                        docker ps -q --filter "name=${container}" | xargs -r docker stop
                        docker ps -a -q --filter "name=${container}" | xargs -r docker rm
                        docker run -d --name ${container} -p ${port}:3000 ${env.IMAGE_NAME}
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
                        docker tag ${env.IMAGE_NAME} $DOCKER_USER/${env.IMAGE_NAME}
                        docker push $DOCKER_USER/${env.IMAGE_NAME}
                    """
                }
            }
        }
    }
}
