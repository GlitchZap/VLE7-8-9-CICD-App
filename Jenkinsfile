
pipeline {
    agent any

    environment {
        IMAGE_NAME = "vle7-app"
        IMAGE_TAG = "v${BUILD_NUMBER}"
        CLUSTER_NAME = "vle-cluster"
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out application source code'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('Test Docker Image') {
            steps {
                sh '''
                    docker rm -f vle7-ci-test || true
                    docker run -d --name vle7-ci-test -p 8086:80 ${IMAGE_NAME}:${IMAGE_TAG}
                    sleep 3
                    curl -f http://localhost:8086
                    docker rm -f vle7-ci-test
                '''
            }
        }

        stage('Load Image into Kubernetes') {
            steps {
                sh 'kind load docker-image ${IMAGE_NAME}:${IMAGE_TAG} --name ${CLUSTER_NAME}'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl set image deployment/vle7-deployment \
                    vle7-container=${IMAGE_NAME}:${IMAGE_TAG} \
                    --kubeconfig=${KUBECONFIG}

                    kubectl rollout status deployment/vle7-deployment \
                    --kubeconfig=${KUBECONFIG}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'kubectl get pods --kubeconfig=${KUBECONFIG}'
                sh 'kubectl get services --kubeconfig=${KUBECONFIG}'
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }
        failure {
            echo 'CI/CD Pipeline failed. Check the console output.'
        }
    }
}
