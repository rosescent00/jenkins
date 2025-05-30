---
# Jenkinsfile - Jenkins 자동 배포 파이프라인 (Maven 사용)
# ----------------------------------------------------------
pipeline {
  agent {
    docker {
      image 'maven:3.9.2-eclipse-temurin-17'
    }
  }

  environment {
    DOCKER_IMAGE = "your-dockerhub-id/spring-app:latest"
    DOCKER_CRED = 'dockerhub-creds'
    KUBECONFIG_CRED = 'kubeconfig-creds'
  }

  stages {
    stage('Clone') {
      steps {
        git credentialsId: 'github-creds', url: 'https://github.com/yourname/your-repo.git'
      }
    }

    stage('Build') {
      steps {
        sh 'mvn clean package -DskipTests'
      }
    }

    stage('Docker Build & Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_CRED}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh """
            docker build -t $DOCKER_IMAGE .
            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
            docker push $DOCKER_IMAGE
          """
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        withCredentials([file(credentialsId: "${KUBECONFIG_CRED}", variable: 'KUBECONFIG')]) {
          sh 'kubectl --kubeconfig=$KUBECONFIG apply -f k8s/deployment.yaml'
        }
      }
    }
  }
}
