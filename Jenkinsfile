pipeline {
  agent any

  environment {
    DOCKER_IMAGE = "rosescent00/spring-app:latest"
    DOCKER_CRED = 'dockerhub-creds'
    KUBECONFIG_CRED = 'kubeconfig-creds'
  }

  stages {
    stage('Clone') {
      steps {
        git credentialsId: 'jenkins-token', url: 'https://github.com/rosescent00/spring-app.git'
      }
    }

    stage('Install Maven') {
      steps {
        sh '''
          apt update && apt install -y maven
        '''
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
