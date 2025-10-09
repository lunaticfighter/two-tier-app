pipeline {

  agent any

  environment {

    DOCKER_USRNAME = 'jspunit00@gmail.com'
    //GIT_COMMIT_SHORT = 'env.GIT_COMMIT_SHORT'
  }

  stages {

    stage('Scm Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Set Environment Variables') {
      steps {
        script {
          env.GIT_COMMIT_SHORT = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim() //with env. It is global — available in all stages of the pipeline.
          echo "The current Build Tag is: ${env.BUILD_TAG}"

        }
      }
    }

    stage('Build Docker image') {
      steps {
        script {
          sh " docker build -t jspunit00/two-tier-app:${env.GIT_COMMIT_SHORT} ."
        }
      }
    }

    stage('Push the Docker Image to Docker Hub') {
      steps {
        script {
          withCredentials([string(credentialsId: 'dockercreds', variable: 'dockercreds')]) {
            sh "docker login -u ${DOCKER_USRNAME} -p ${docketcreds}"
          }
          sh "docker push jspunit00/docker-sample:${env.GIT_COMMIT_SHORT}"
        }
      }

    }

  }

}
