pipeline {
    agent any
    environment {
        NODE_ENV = 'production'
        PORT = ''
        DOCKER_IMAGE = ''
        LOGO_PATH = ''
    }

    tools {
        git 'Default'  // This assumes that 'Default' is a named Git tool configuration in Jenkins. Ensure it is configured for Linux.
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'npm install'  // Using `sh` instead of `bat`
            }
        }
        stage('Test') {
            steps {
                sh 'npm test'  // Using `sh` instead of `bat`
            }
        }
        stage('Docker Build') {
            environment {
                PORT = ''
                DOCKER_IMAGE = ''
                LOGO_PATH = ''
            }
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        PORT = '3000'
                        DOCKER_IMAGE = 'nodemain:v1.0'
                        LOGO_PATH = 'src/logo.svg'  // Linux uses forward slashes for paths
                    } else if (env.BRANCH_NAME == 'dev') {
                        PORT = '3001'
                        DOCKER_IMAGE = 'nodedev:v1.0'
                        LOGO_PATH = 'src/logo.svg'
                    }
                    echo "LOGO_PATH: ${LOGO_PATH}"
                    echo "Checking if the logo file exists..."
                    sh "test -f ${LOGO_PATH} && echo File exists || (echo File not found && exit 1)"
                    sh "cp ${LOGO_PATH} public/logo.svg"
                    echo "Checking Docker status..."
                    sh "docker --version"
                    sh "docker info"
                    echo "Building Docker image..."
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    echo "Stopping and removing any existing containers..."
                    sh """
                    docker ps -q --filter "ancestor=${DOCKER_IMAGE}" | xargs --no-run-if-empty docker stop || echo No container to stop
                    docker ps -a -q --filter "ancestor=${DOCKER_IMAGE}" | xargs --no-run-if-empty docker rm || echo No container to remove
                    """
                    echo "Running the Docker container..."
                    if (env.BRANCH_NAME == 'main') {
                        sh "docker run -d --expose 3000 -p 3000:3000 ${DOCKER_IMAGE}"
                    } else if (env.BRANCH_NAME == 'dev') {
                        sh "docker run -d --expose 3001 -p 3001:3000 ${DOCKER_IMAGE}"
                    }
                }
            }
        }
    }
}