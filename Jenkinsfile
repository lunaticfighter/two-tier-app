pipeline {

  agent any

  parameters {
    string(name: 'GIT_REPO_URL', description: 'URL of the Git repository containing the Helm chart', defaultValue: "https://github.com/lunaticfighter/two-tier-app-devops.git")
    string(name: 'REPO_FOLDER_NAME', description: 'Folder name', defaultValue: 'two-tier-app-devops')
  }

  environment {
    DOCKER_USRNAME = 'jspunit00@gmail.com'
  }

  stages {

    stage('SCM Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Set Environment Variables') {
      steps {
        script {
          env.GIT_COMMIT_SHORT = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
          echo "The current Build Tag is: ${env.BUILD_TAG}"
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          sh "docker build -t jspunit00/two-tier-app:${env.GIT_COMMIT_SHORT} ."
        }
      }
    }

    stage('Push Docker Image to Docker Hub') {
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: 'dockercreds', variable: 'dockercreds')]) {
            sh "docker login -u ${DOCKER_USRNAME} -p ${dockercreds}"
          }
          sh "docker push jspunit00/two-tier-app:${env.GIT_COMMIT_SHORT}"
        }
      }
    }

    stage('SCM Checkout 2') {
      steps {
        echo 'Checking out SCM again...'
        checkout scm
      }
    }

    stage('Update Helm Chart') {
      steps {
        script {

          dir("${params.REPO_FOLDER_NAME}") {
            // Checkout Helm chart repo
            checkout scmGit(
              branches: [[name: '*/main']],
              userRemoteConfigs: [[
                credentialsId: 'lunaticfighter',
                url: "${params.GIT_REPO_URL}"
              ]]
            )
          }

          echo "Deploying to environment using image tag: ${env.GIT_COMMIT_SHORT}"

          dir("${params.REPO_FOLDER_NAME}") {
            // Update Helm chart values.yaml
            def valuesFile = readFile('values.yaml')
            def updatedValues = valuesFile.replaceAll(/tag: .*/, "tag: ${env.GIT_COMMIT_SHORT}")
            writeFile file: 'values.yaml', text: updatedValues
          }

          dir("${params.REPO_FOLDER_NAME}") {
            // Git config and commit updated values.yaml
            sh "git config --global --add safe.directory '*'"
            sh "git config --global user.email 'jspunit00@gmail.com'"
            sh "git config --global user.name 'punit'"
            sh 'git add .'
            sh 'git commit -m "Update image tag"'

            // Push to GitHub (token should be stored securely)
            sh 'git push https://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@github.com/lunaticfighter/two-tier-app-devops.git HEAD:main'
          }

        }
      }
    }

  }
}
