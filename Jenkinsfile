pipeline {
    environment {
        imagename = "andreavomero99/ciao"
        registryCredential = 'DockerHub'
        dockerImage = ''
        BRANCH_NAME = ''
        GIT_TAG = ''
    }
    agent any
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
                    BRANCH_NAME = sh(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()
                    GIT_TAG = sh(script: 'git describe --tags --exact-match || echo ""', returnStdout: true).trim()
                    echo "Cloned Branch: ${BRANCH_NAME}"
                    echo "Git Tag: ${GIT_TAG}"
                }
            }
        }
        stage('Debug Info') {
            steps {
                script {
                    echo "Branch Name: ${BRANCH_NAME}"
                    echo "Git Commit: ${env.GIT_COMMIT}"
                    echo "Git Tag: ${GIT_TAG}"
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
                    def tag = ""
                    if (GIT_TAG) {
                        tag = GIT_TAG
                    } else if (BRANCH_NAME == 'main') {
                        tag = 'latest'
                    } else if (BRANCH_NAME == 'secondary') {
                        tag = "secondary-${env.GIT_COMMIT}"
                    } else {
                        tag = "${BRANCH_NAME}-${env.GIT_COMMIT}"
                    }
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push(tag)
                    }
                }
            }
        }
        stage('Remove Unused docker image') {
            steps {
                script {
                    sh "docker rmi ${imagename}:${env.GIT_COMMIT}"
                    if (GIT_TAG) {
                        sh "docker rmi ${imagename}:${GIT_TAG}"
                    } else if (BRANCH_NAME == 'main') {
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


