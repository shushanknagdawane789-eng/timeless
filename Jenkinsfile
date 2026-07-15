pipeline {
    agent any

    environment {
        DOCKER_REPO = "shushanknagdawane789"
        DOCKER_USER = "timeless"
        IMAGE_NAME = "timeless"
        CONTAINER_NAME = "timeless-container"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/shushanknagdawane789-eng/timeless.git'
            }
        }

        stage('Verify Environment') {
            steps {
                sh '''
                echo "Java Version:"
                java -version

                echo "Maven Version:"
                mvn -version

                echo "Docker Version:"
                docker --version
                '''
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t ${DOCKER_REPO}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-hub-creds',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin'
                }
            }
        }

        stage('Docker Push') {
            steps {
                sh '''
                docker tag ${DOCKER_REPO}:${BUILD_NUMBER} ${DOCKER_REPO}/${DOCKER_USER}:${BUILD_NUMBER}
                docker push ${DOCKER_REPO}/${DOCKER_USER}:${BUILD_NUMBER}
                '''
            }
        }

        stage('K8s-Deployment') {
            steps {
                sh '''
                sed -i "s|shushanknagdawane789/timeless:latest|${DOCKER_REPO}/${DOCKER_USER}:${BUILD_NUMBER}|g" k8s/deployment.yaml
                cat k8s/deployment.yaml
                '''
            }
        }
    }
}