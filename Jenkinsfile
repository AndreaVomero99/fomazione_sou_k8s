pipeline {
  environment {
    imagename = "andreavomero99/ciao"
    registryCredential = 'DockerHub'
    dockerImage = ''
    BRANCH_NAME = ''
  }
  agent any
  stages {
    stage('Debug Info') {
      steps {
        script {
          echo "Branch Name: ${env.BRANCH_NAME}"
          echo "Git Commit: ${env.GIT_COMMIT}"

    stage('Cloning Git') {
      steps {
        git([url: 'https://github.com/AndreaVomero99/fomazione_sou_k8s', branch: 'secondary', credentialsId: 'GitHub'])
        script {
          BRANCH_NAME = sh(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()
        }
      }
    }
    stage('Debug Info') {
      steps {
        script {
          echo "Branch Name: ${BRANCH_NAME}"
          echo "Git Commit: ${env.GIT_COMMIT}"
        }
      }
    }
    stage('Building image') {
      steps {
        script {
          dockerImage = docker.build("${imagename}:${env.GIT_COMMIT}")
        }
      }
    }
    stage('Deploy Image') {
      steps {
        script {
          docker.withRegistry('', registryCredential) {
            dockerImage.push()
            if (BRANCH_NAME == 'main') {
              dockerImage.push('latest')
            } else if (BRANCH_NAME == 'secondary') {
              dockerImage.push("secondary-${env.GIT_COMMIT}")
            } else {
              dockerImage.push("${BRANCH_NAME}-${env.GIT_COMMIT}")
            }
          }
        }
      }
    }
    stage('Remove Unused docker image') {
      steps {
        script {
          sh "docker rmi ${imagename}:${env.GIT_COMMIT}"
          if (BRANCH_NAME == 'main') {
            sh "docker rmi ${imagename}:latest"
          } else if (BRANCH_NAME == 'secondary') {
            sh "docker rmi ${imagename}:secondary-${env.GIT_COMMIT}"
          } else {
            sh "docker rmi ${imagename}:${BRANCH_NAME}-${env.GIT_COMMIT}"
          }
        }
      }
    }
  }
  post {
    always {
      cleanWs()
    }
  }
}

