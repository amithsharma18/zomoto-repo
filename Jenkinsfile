pipeline {
    agent any

    environment {
        NODE_ENV = 'production'
        IMAGE_NAME = 'amith1808/zomoto-hub/zomato:v1'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/amithsharma18/zomoto-repo.git'
            }
        }

        stage('Check Versions') {
            steps {
                sh '''
                    echo "Java Version:"
                    java -version

                    echo "Node Version:"
                    node -v

                    echo "NPM Version:"
                    npm -v

                    echo "Docker Version:"
                    docker --version
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('zomato-app') {
                    sh 'npm install'
                }
            }
        }

        stage('Build Application') {
            environment {
                NODE_OPTIONS = '--openssl-legacy-provider'
                CI = 'false'
            }
            steps {
                dir('zomato-app') {
                    sh 'npm run build'
                }
            }
        }

        stage('Archive Build') {
            steps {
                archiveArtifacts artifacts: 'zomato-app/build/**', fingerprint: true
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('zomato-app') {
                    sh '''
                        docker build -t ${IMAGE_NAME} .
                    '''
                }
            }
        }

        stage('Docker Hub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    docker push ${IMAGE_NAME}
                '''
            }
        }
    }

    post {
        success {
            echo '✅ React application built successfully.'
            echo '✅ Docker image pushed to Docker Hub successfully.'
        }

        failure {
            echo '❌ Pipeline failed.'
        }

        always {
            cleanWs()
        }
    }
}
