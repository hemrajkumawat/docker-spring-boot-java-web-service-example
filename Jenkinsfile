pipeline {
    agent any
    environment {
        REGISTRY = 'localhost:5000'
        IMAGE_NAME = 'docker-java-hello-world-app'
        IMAGE_TAG = 'latest'
        CONTAINER_NAME = 'registry'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/hemrajkumawat/docker-spring-boot-java-web-service-example.git'
            }
        }
        stage('Build Image') {
            steps {
                script {
                    sh 'docker build -t $REGISTRY/$IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    // Run tests inside the container
                    sh 'docker run --rm  -v $(pwd):/app -w /app -e JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk $REGISTRY/$IMAGE_NAME:$IMAGE_TAG mvn test'
                }
            }
        }
        stage('Push to Registry') {
            steps {
                script {
                    sh 'docker push $REGISTRY/$IMAGE_NAME:$IMAGE_TAG'
                }
            }
        }
        stage('Security Scan - Trivy') {
            steps {
                script {
                    sh 'docker run --rm aquasec/trivy image $REGISTRY/$IMAGE_NAME:$IMAGE_TAG'
                }
            }
        }
        stage('Deploy Locally') {
            steps {
                script {
                    // Stop existing container if running
                    sh 'docker stop $CONTAINER_NAME || true && docker rm $CONTAINER_NAME || true'
                    // Run new container
                    sh 'docker run -d -p 8080:8080 --name $CONTAINER_NAME $REGISTRY/$IMAGE_NAME:$IMAGE_TAG'
                }
            }
        }
    }
    post {
        always {
            sh 'docker system prune -f'
        }
        failure {
            echo "Build failed, check logs for details."
            //mail to: 'admin@example.com', subject: 'Build Failed', body: 'Check Jenkins logs for details.'
        }
    }
}
