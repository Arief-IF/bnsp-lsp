pipeline {
    agent any

    environment {
        // Menggunakan variabel lingkungan Anda
        DOCKERHUB_USER = "arie23262ti"
        IMAGE_NAME     = "bnsp-lsp" // Menggunakan nama yang benar dari build sebelumnya
        
        // Target Deployment EC2
        EC2_HOST       = "16.176.231.5" // Ganti dengan IP Publik EC2 Anda
        EC2_USER       = "ubuntu"       // User OS EC2 Anda
        EC2_PORT       = 8080           // Port publik yang ingin Anda gunakan di EC2
    }

    stages {
        stage('Checkout & Setup') {
            steps {
                echo '1. Checkout kode dari SCM'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                // Image akan dilabeli: arie23262ti/bnsp-lsp:latest
                sh "docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:latest ."
            }
        }

        stage('Login & Push Image') {
            steps {
                // Gunakan kredensial Docker Hub
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-cred', // Pastikan ID ini benar di Jenkins Anda
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    // 1. Login
                    sh "echo ${PASS} | docker login -u ${USER} --password-stdin"
                    
                    // 2. Push Image ke Docker Hub
                    sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest"
                    
                    // Logout dari Jenkins Agent (praktik baik keamanan)
                    sh "docker logout"
                }
            }
        }

        stage('Deploy ke AWS EC2 via SSH') {
            steps {
                // Kunci Rahasia SSH harus diakses melalui withCredentials
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'ec2-ssh-key-id', // ID Kunci SSH Anda di Jenkins
                    keyFileVariable: 'SSH_KEY'      // Variabel yang menampung path ke kunci
                )]) {
                    // Perintah gabungan yang dieksekusi di EC2 melalui SSH
                    sh """
                        echo 'Deploying to ${EC2_HOST}...'
                        
                        # Command dikirim ke EC2 menggunakan kunci privat
                        ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << EOF
                            
                            # 1. Tarik image terbaru dari Docker Hub
                            sudo docker pull ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                            
                            # 2. Hentikan dan hapus container lama (idempoten)
                            sudo docker stop ${IMAGE_NAME} || true
                            sudo docker rm ${IMAGE_NAME} || true
                            
                            # 3. Jalankan container baru (Mapping ke Port EC2)
                            sudo docker run -d -p ${EC2_PORT}:80 --name ${IMAGE_NAME} ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                            
                            # 4. Verifikasi
                            sudo docker ps | grep ${IMAGE_NAME}
                        EOF
                    """
                }
            }
        }
    }

    post {
        success {
            echo "SUKSES: Deployment ke ${EC2_HOST} selesai. Website dapat diakses di Port ${EC2_PORT}."
        }
        failure {
            echo 'Pipeline gagal! Cek log build dan koneksi SSH EC2.'
        }
    }
}
