pipeline {
    environment {
        imagename = "andreavomero99/ciao"
        registryCredential = 'DockerHub'
        dockerImage = ''
        GIT_TAG = ''
    }
    agent any
    parameters {
        string(name: 'TAG', defaultValue: '', description: 'Custom tag for Docker image')
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Cloning Git') {
            steps {
                script {
                    git branch: 'secondary', credentialsId: 'GitHub', url: 'https://github.com/AndreaVomero99/fomazione_sou_k8s'
                    GIT_TAG = sh(script: 'git describe --tags --abbrev=0', returnStdout: true).trim()
                    echo "Git Tag: ${GIT_TAG}"
                }
            }
        }
        stage('Building image') {
            steps {
                script {
                    if (params.TAG) {
                        // Se è stato specificato un tag personalizzato, utilizza quello
                        GIT_TAG = params.TAG
                    } else {
                        // Altrimenti, utilizza il nome del branch seguito dall'ID del commit
                        GIT_TAG = "${GIT_TAG}-${env.GIT_COMMIT}"
                    }
                    dockerImage = docker.build("${imagename}:${GIT_TAG}")
                }
            }
        }
        stage('Deploy Image') {
            steps {
                script {
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push()
                        dockerImage.push("${GIT_TAG}")
                    }
                }
            }
        }
        stage('Remove Unused docker image') {
            steps {
                script {
                    sh "docker rmi ${imagename}:${GIT_TAG}"
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
