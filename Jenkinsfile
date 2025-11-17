pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        DOCKERHUB_USER = "arie23262ti"
        IMAGE_NAME     = "bnsp-lsp"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Static HTML - tidak ada proses build.'
            }
        }

        stage('Test') {
            steps {
                echo 'Listing file'
                sh 'ls -lah'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:latest .
                """
            }
        }

        stage('Login Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-cred',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh "echo ${PASS} | docker login -u ${USER} --password-stdin"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh """
                    docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                """
            }
        }

        stage('Deploy Container') {
            steps {
                sh """
                    docker stop bnsp || true
                    docker rm bnsp || true
                    docker pull ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                    docker run -d -p 80:80 --name bnsp ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                """
            }
        }
    }

    post {
        success {
            echo 'SUKSES: Docker Hub & Container sudah terupdate!'
        }
        failure {
            echo 'Pipeline gagal!'
        }
    }
}