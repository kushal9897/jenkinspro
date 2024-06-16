pipeline {
    agent { 
        docker { image 'node:14-alpine' } 
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials-id')
        IMAGE_NAME = 'my-nodejs-app'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/yourusername/nodejs-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${env.IMAGE_NAME}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    docker.image("${env.IMAGE_NAME}:${env.BUILD_NUMBER}").inside {
                        sh 'npm install'
                        sh 'npm test'
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'DOCKERHUB_CREDENTIALS') {
                        docker.image("${env.IMAGE_NAME}:${env.BUILD_NUMBER}").push()
                    }
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                script {
                    sh '''
                    docker pull ${env.IMAGE_NAME}:${env.BUILD_NUMBER}
                    docker stop staging-container || true
                    docker rm staging-container || true
                    docker run -d --name staging-container -p 8080:8080 ${env.IMAGE_NAME}:${env.BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Optional: Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                script {
                    sh '''
                    docker pull ${env.IMAGE_NAME}:${env.BUILD_NUMBER}
                    docker stop production-container || true
                    docker rm production-container || true
                    docker run -d --name production-container -p 80:8080 ${env.IMAGE_NAME}:${env.BUILD_NUMBER}
                    '''
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        failure {
            mail to: 'team@example.com',
                 subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
                 body: "Something is wrong with ${env.JOB_NAME} ${env.BUILD_NUMBER}. Please check the logs."
        }
    }
}

