pipeline {
    agent any

    environment {
        GIT_URL               = 'https://github.com/lpintoamsys/kubernetescode.git'
        GIT_CREDENTIALS       = 'github-credentials'
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'
        DOCKERHUB_REPO        = 'kubernetescode'
        IMAGE_TAG             = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: GIT_URL, credentialsId: GIT_CREDENTIALS]]])
            }
        }

        stage('Build') {
            steps {
                sh "docker build -t ${DOCKERHUB_REPO}:${IMAGE_TAG} ."
                sh "docker tag ${DOCKERHUB_REPO}:${IMAGE_TAG} ${DOCKERHUB_REPO}:latest"
            }
        }

        stage('Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS, usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                    sh 'echo $DH_PASS | docker login -u $DH_USER --password-stdin'
                    sh "docker push ${DOCKERHUB_REPO}:${IMAGE_TAG}"
                    sh "docker push ${DOCKERHUB_REPO}:latest"
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
