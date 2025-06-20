pipeline {
    agent any

    tools {
        nodejs 'nodejs'
    }

    environment {
        DOCKER_IMAGE = ''
        CONTAINER_NAME = ''
    }

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'github-ssh', url: 'https://github.com/giotsintsadze/cicd-pipeline', branch: "${env.BRANCH_NAME}"
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.DOCKER_IMAGE = 'nodemain:v1.0'
                        env.CONTAINER_NAME = 'main_app'
                    } else if (env.BRANCH_NAME == 'dev') {
                        env.DOCKER_IMAGE = 'nodedev:v1.0'
                        env.CONTAINER_NAME = 'dev_app'
                    } else {
                        error("Unsupported branch: ${env.BRANCH_NAME}")
                    }
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Clean Previous Container') {
            steps {
                script {
                    sh """
                        if [ \$(docker ps -q -f name=${CONTAINER_NAME}) ]; then
                            docker stop ${CONTAINER_NAME}
                            docker rm ${CONTAINER_NAME}
                        fi
                    """
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    def port = env.BRANCH_NAME == 'main' ? '3000:3000' : '3001:3000'
                    sh "docker run -d --name ${CONTAINER_NAME} --expose ${port.split(':')[0]} -p ${port} ${DOCKER_IMAGE}"
                }
            }
        }
    }
}
