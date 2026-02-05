pipeline {
  agent none

  environment {
    DOCKER_IMAGE = "lpintoamsys/kubernetescode"
    IMAGE_TAG    = "${BUILD_NUMBER}"
  }

  stages {

    stage('Checkout') {
      agent any
      steps {
        checkout scm
      }
    }

    stage('Build & Push Image (Kaniko)') {
      agent {
        kubernetes {
          yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: kaniko-builder
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
          echo "🚀 Starting Kaniko build"
          /kaniko/executor \
            --dockerfile=Dockerfile \
            --context=dir:///workspace \
            --destination=${DOCKER_IMAGE}:${IMAGE_TAG} \
            --destination=${DOCKER_IMAGE}:latest \
            --cache=true \
            --cache-repo=${DOCKER_IMAGE}-cache
          """
        }
      }
    }
  }

  post {
    success {
      echo "✅ Image pushed successfully"
    }
    failure {
      echo "❌ Build failed"
    }
  }
}
