pipeline {
    agent any

    environment {
        GIT_URL               = 'https://github.com/lpintoamsys/kubernetescode.git'
        GIT_CREDENTIALS       = 'github-credentials'
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'
        DOCKERHUB_USERNAME    = 'lpintoamsys'
        DOCKERHUB_REPO        = 'kubernetescode'
        DOCKER_IMAGE          = "${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO}"
        KUBE_CONFIG_CREDENTIALS = 'kubeconfig'
        KUBE_NAMESPACE        = 'default'
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
                        sh "podman build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
                        sh "podman tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_IMAGE}:latest"
                    }
                }

            stage('Push') {
                steps {
                    withCredentials([
                        usernamePassword(
                            credentialsId: DOCKERHUB_CREDENTIALS,
                            usernameVariable: 'DH_USER',
                            passwordVariable: 'DH_PASS'
                        )
                    ]) {
                        sh 'echo $DH_PASS | podman login docker.io -u $DH_USER --password-stdin'
                        sh "podman push ${DOCKER_IMAGE}:${IMAGE_TAG}"
                        sh "podman push ${DOCKER_IMAGE}:latest"
                    }
                }
            }

            stage('Deploy') {
                steps {
                    withCredentials([file(credentialsId: KUBE_CONFIG_CREDENTIALS, variable: 'KUBECONFIG')]) {
                        sh "kubectl apply -f deployment.yaml -n ${KUBE_NAMESPACE}"
                        sh "kubectl set image deployment/kubernetescode app=${DOCKER_IMAGE}:${IMAGE_TAG} -n ${KUBE_NAMESPACE}"
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
