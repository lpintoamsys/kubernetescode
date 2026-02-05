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
    image: gcr.io/kaniko-project/executor:v1.23.2
    imagePullPolicy: Always
    command:
    - /kaniko/executor
    args:
    - --dockerfile=Dockerfile
    - --context=dir:///workspace
    - --destination=${DOCKER_IMAGE}:${IMAGE_TAG}
    - --destination=${DOCKER_IMAGE}:latest
    - --cache=true
    - --cache-repo=${DOCKER_IMAGE}-cache
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
          sh 'echo "Building and pushing image with Kaniko..."'
        }
      }
    }
  }

  post {
    success {
      echo "✅ Image pushed successfully: ${DOCKER_IMAGE}:${IMAGE_TAG}"
    }
    failure {
      echo "❌ Image build failed"
    }
  }
}
