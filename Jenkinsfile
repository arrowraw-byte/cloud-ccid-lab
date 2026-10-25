
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'aaravarora/cloud-ccid-lab'
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/arrowraw-byte/cloud-ccid-lab.git'
            }
        }

        stage('Test') {
            steps {
                sh 'python3 -m py_compile app.py'
                echo 'Python syntax check passed.'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                      -t ${IMAGE_NAME}:${BUILD_NUMBER} \
                      -t ${IMAGE_NAME}:latest .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKERHUB_USER',
                    passwordVariable: 'DOCKERHUB_TOKEN'
                )]) {
                    sh '''
                        set +x
                        printf '%s' "$DOCKERHUB_TOKEN" |
                          docker login -u "$DOCKERHUB_USER" --password-stdin

                        docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                        docker push ${IMAGE_NAME}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                sh '''
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml

                    kubectl set image deployment/cloud-ccid-app \
                      flask-app=${IMAGE_NAME}:${BUILD_NUMBER}

                    kubectl rollout status deployment/cloud-ccid-app \
                      --timeout=180s

                    kubectl get deployments
                    kubectl get pods -o wide
                    kubectl get services
                '''
            }
        }

        stage('Verify Application') {
            steps {
                sh '''
                    kubectl run ci-curl-check \
                      --image=curlimages/curl:latest \
                      --restart=Never \
                      --rm -i \
                      --command -- \
                      curl --fail --retry 10 --retry-connrefused \
                      --retry-delay 3 \
                      http://cloud-ccid-service:5000/health
                '''
            }
        }
    }
}
