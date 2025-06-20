pipeline {
    agent any

    tools {
        nodejs 'nodejs'
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
                    def image = ''
                    def container = ''
                    if (env.BRANCH_NAME == 'main') {
                        image = 'nodemain:v1.0'
                        container = 'main_app'
                    } else if (env.BRANCH_NAME == 'dev') {
                        image = 'nodedev:v1.0'
                        container = 'dev_app'
                    } else {
                        error("Unsupported branch: ${env.BRANCH_NAME}")
                    }

                    sh "docker build -t ${image} ."

                    // Save container name/image for next stages using environment
                    env.CONTAINER_NAME = container
                    env.DOCKER_IMAGE = image
                }
            }
        }

        stage('Clean Previous Container') {
            steps {
                script {
                    def container = env.CONTAINER_NAME
                    sh """
                        if [ \$(docker ps -q -f name=${container}) ]; then
                            docker stop ${container}
                            docker rm ${container}
                        fi
                    """
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    def port = env.BRANCH_NAME == 'main' ? '3000:3000' : '3001:3000'
                    def container = env.CONTAINER_NAME
                    def image = env.DOCKER_IMAGE
                    sh "docker run -d --name ${container} --expose ${port.split(':')[0]} -p ${port} ${image}"
                }
            }
        }
    }
}
