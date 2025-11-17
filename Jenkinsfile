pipeline {
    agent any

    triggers {
        githubPush()
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
                echo 'Menjalankan pseudo-test (untuk demo).'
                sh 'ls -lah'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploy ke Apache /var/www/html'
                sh """
                    rm -rf /var/www/html/*
                    cp -r * /var/www/html/
                """
            }
        }
    }

    post {
        success {
            echo 'Pipeline sukses. Web sudah terupdate.'
        }
        failure {
            echo 'Pipeline gagal. Cek log di Console Output.'
        }
    }
}