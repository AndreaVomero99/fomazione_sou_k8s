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
                    git branch: 'main', credentialsId: 'GitHub', url: 'https://github.com/AndreaVomero99/fomazione_sou_k8s'
                    BRANCH_NAME = env.GIT_BRANCH
                    GIT_TAG = sh(script: 'git tag --points-at HEAD | head -n 1', returnStdout: true).trim()
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
                        dockerImage.push(tag)
                        if (BRANCH_NAME == 'main') {
                            dockerImage.push('latest')
                        }
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
                    def tag = ""
                    if (GIT_TAG) {
                        tag = GIT_TAG
                        if (BRANCH_NAME == 'main') {
                            sh "docker rmi ${imagename}:${tag}"
                        }
                    } else if (BRANCH_NAME == 'main') {
                        tag = 'latest'
                    } else if (BRANCH_NAME == 'secondary') {
                        tag = "secondary-${env.GIT_COMMIT}"
                    } else {
                        tag = "${BRANCH_NAME}-${env.GIT_COMMIT}"
                    }
                    sh "docker rmi ${imagename}:${env.GIT_COMMIT}"
                    sh "docker rmi ${imagename}:${tag}"
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


