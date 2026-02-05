pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'
        DOCKERHUB_USERNAME    = 'lpintoamsys'
        DOCKERHUB_REPO        = 'hello-docker'
        IMAGE_TAG             = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Build') {
            steps {
                sh "docker build -t ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:${IMAGE_TAG} ."
                sh "docker tag ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:${IMAGE_TAG} ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:latest"
            }
        }

        stage('Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS, usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                    sh 'echo $DH_PASS | docker login -u $DH_USER --password-stdin'
                    sh "docker push ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:${IMAGE_TAG}"
                    sh "docker push ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}:latest"
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
    }
}
