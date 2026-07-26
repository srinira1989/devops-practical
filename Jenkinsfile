pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'srinira'
        IMAGE_NAME = 'devops-flask'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Docker Build') {
            steps {
                bat """
                    docker build -t %DOCKERHUB_USERNAME%/%IMAGE_NAME%:%IMAGE_TAG% .
                """
            }
        }

    }
}