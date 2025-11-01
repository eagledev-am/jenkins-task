pipeline {
    agent any

    tools {
        maven 'mvn-3-5-2'
        jdk 'jdk11'
    }

    environment {
        IMAGE_NAME = "abdomagdy/jenkins-app"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/eagledev-am/jenkins-task.git'
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "Build Number: ${currentBuild.number}"
                    // if (currentBuild.number < 5) {
                    //     error("Stopping pipeline!")  // Remove after testing
                    // }
                }
                sh "mvn clean package"
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub-pass', variable: 'DOCKERHUB_PASS')]) {
                    sh '''
                    echo "$DOCKERHUB_PASS" | docker login -u abdomagdy --password-stdin
                    docker info | grep Username
                    '''
                }
            }
        }

        stage('Docker Push') {
            steps {
                sh """
                docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
                docker push ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Deploy') {
            steps {
                sh """
                echo "Deploying container..."
                docker stop jenkins-app || true
                docker rm jenkins-app || true
                docker run -d -p 9000:8080 --name jenkins-app ${IMAGE_NAME}:${BUILD_NUMBER}
                echo "App is running on: http://<your-server-ip>:9000"
                """
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully"
        }
        failure {
            echo "Pipeline failed"
        }
        always {
            echo "Cleaning workspace..."
            cleanWs()
        }
    }
}
