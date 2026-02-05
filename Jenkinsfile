pipeline {
  agent none

  environment {
    DOCKER_IMAGE = "lpintoamsys/kubernetescode"
    IMAGE_TAG = "${BUILD_NUMBER}"
  }

  stages {

    stage('Build & Push (Kaniko)') {
      agent {
        kubernetes {
          yaml """
apiVersion: v1
kind: Pod
spec:
  restartPolicy: Never
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
      - /busybox/sh
      - -c
      - cat
    tty: true
    volumeMounts:
    - name: docker-config
      mountPath: /kaniko/.docker
  volumes:
  - name: docker-config
    secret:
      secretName: dockerhub-secret
"""
        }
      }

      steps {
        container('kaniko') {
          sh """
          echo "Running on agent:"
          hostname
          pwd

          /kaniko/executor \
            --dockerfile=Dockerfile \
            --context=dir:///workspace \
            --destination=${DOCKER_IMAGE}:${IMAGE_TAG} \
            --destination=${DOCKER_IMAGE}:latest
          """
        }
      }
    }
  }
}
