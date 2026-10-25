
pipeline {
    agent any

    stages {
        stage('Clone') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/arrowraw-byte/cloud-ccid-lab.git'
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t cloud-ccid-lab:latest .'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker rm -f cloud-ccid-app || true
                    docker run -d --name cloud-ccid-app \
                        --restart unless-stopped \
                        -p 5000:5000 cloud-ccid-lab:latest
                '''
            }
        }
    }
}
