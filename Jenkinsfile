pipeline {
  environment {
    imagename = "andreavomero99/ciao"
    registryCredential = 'DockerHub'
    dockerImage = ''
  }
  agent any
  stages {
    stage('Cloning Git') {
      steps {
        git([url: 'https://github.com/AndreaVomero99/fomazione_sou_k8s', branch: 'secondary', credentialsId: 'GitHub'])
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
            if (env.BRANCH_NAME == 'main') {
              dockerImage.push('latest')
            } else if (env.BRANCH_NAME == 'secondary') {
              dockerImage.push("secondary-${env.GIT_COMMIT}")
            } else {
              dockerImage.push("${env.BRANCH_NAME}-${env.GIT_COMMIT}")
            }
          }
        }
      }
    }
    stage('Remove Unused docker image') {
      steps {
        script {
          sh "docker rmi ${imagename}:${env.GIT_COMMIT}"
          if (env.BRANCH_NAME == 'main') {
            sh "docker rmi ${imagename}:latest"
          } else if (env.BRANCH_NAME == 'secondary') {
            sh "docker rmi ${imagename}:secondary-${env.GIT_COMMIT}"
          } else {
            sh "docker rmi ${imagename}:${env.BRANCH_NAME}-${env.GIT_COMMIT}"
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
