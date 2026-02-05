pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
spec:
  restartPolicy: Never
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:v1.23.2-debug
    command:
      - /busybox/sh
      - -c
      - cat
    tty: true
  - name: kubectl
    image: dtzar/helm-kubectl:3.14.0
    command:
      - /bin/sh
      - -c
      - cat
    tty: true
"""
    }
  }

  options {
    skipDefaultCheckout(true)
  }

  environment {
    GIT_URL                 = 'https://github.com/lpintoamsys/kubernetescode.git'
    GIT_CREDENTIALS         = 'github-credentials'

    DOCKERHUB_CREDENTIALS   = 'dockerhub-credentials'
    DOCKER_IMAGE            = 'lpintoamsys/kubernetescode'
    IMAGE_TAG               = "${BUILD_NUMBER}"

    KUBE_CONFIG_CREDENTIALS = 'kubeconfig'
    KUBE_NAMESPACE          = 'default'
    DEPLOYMENT_NAME         = 'kubernetescode'
    CONTAINER_NAME          = 'app'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: GIT_URL, credentialsId: GIT_CREDENTIALS]]])
      }
    }

    stage('Build & Push (Kaniko)') {
      steps {
        container('kaniko') {
          withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS, usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
            sh """
              set -eu

              echo \"WORKSPACE=${WORKSPACE}\"
              ls -la

              mkdir -p /kaniko/.docker
              AUTH=\$(printf '%s:%s' \"$DH_USER\" \"$DH_PASS\" | base64 | tr -d '\\n')
              cat > /kaniko/.docker/config.json <<EOF
              {\"auths\":{\"https://index.docker.io/v1/\":{\"auth\":\"$AUTH\"}}}
EOF

              /kaniko/executor \\
                --dockerfile=Dockerfile \\
                --context=dir://${WORKSPACE} \\
                --destination=${DOCKER_IMAGE}:${IMAGE_TAG} \\
                --destination=${DOCKER_IMAGE}:latest
            """
          }
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        container('kubectl') {
          withCredentials([file(credentialsId: KUBE_CONFIG_CREDENTIALS, variable: 'KUBECONFIG')]) {
            sh """
              set -eu
              kubectl version --client=true
              kubectl apply -n ${KUBE_NAMESPACE} -f deployment.yaml
              kubectl set image -n ${KUBE_NAMESPACE} deployment/${DEPLOYMENT_NAME} ${CONTAINER_NAME}=${DOCKER_IMAGE}:${IMAGE_TAG}
              kubectl rollout status -n ${KUBE_NAMESPACE} deployment/${DEPLOYMENT_NAME} --timeout=180s
              kubectl get pods -n ${KUBE_NAMESPACE} -l app=${DEPLOYMENT_NAME} -o wide
            """
          }
        }
      }
    }
  }
}
