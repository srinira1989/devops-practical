pipeline {

    agent any

    environment {
        IMAGE = "srinira/devops-flask"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Build') {
            steps {
                echo 'Checking Python environment'
                sh 'python3 --version'
            }
        }

        stage('Test') {
            steps {
                echo 'Installing dependencies and running tests'
                sh '''
                    pip3 install -r requirements.txt
                    pytest
                '''
            }
        }

        stage('Docker Build') {
            steps {
                echo "Building Docker image ${IMAGE}:${TAG}"

                sh "docker build -t ${IMAGE}:${TAG} ."
                   
            }
        }

        stage('Image Scan') {
            steps {
                echo 'Scanning Docker image for CRITICAL vulnerabilities'

                sh '''
                    trivy image \
                      --exit-code 1 \
                      --severity CRITICAL \
                      ${IMAGE}:${TAG}
                '''
            }
        }

        stage('Push to Registry') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                          -u "$DOCKER_USER" \
                          --password-stdin

                        docker push ${IMAGE}:${TAG}

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {

                echo 'Deploying application to Minikube'

                sh '''
                    kubectl apply -f k8s/namespace.yaml
                    kubectl apply -f k8s/serviceaccount.yaml
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml

                    kubectl set image deployment/flask-app \
                      flask-app=${IMAGE}:${TAG} \
                      -n devops-app

                    kubectl rollout status deployment/flask-app \
                      -n devops-app
                '''
            }
        }
    }

    post {

        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline failed. Check the stage logs.'
        }
    }
}