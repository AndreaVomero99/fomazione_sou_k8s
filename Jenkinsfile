pipeline {
    environment {
        imagename = "andreavomero99/ciao"
        registryCredential = 'DockerHub'
        dockerImage = ''
        BRANCH_NAME = ''
        GIT_TAG = ''
        GIT_COMMIT_HASH = ''
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
                    // Checkout the branch that triggered the build
                    checkout scm
                    BRANCH_NAME = sh(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()
                    GIT_TAG = sh(script: 'git describe --tags --exact-match || echo ""', returnStdout: true).trim()
                    GIT_COMMIT_HASH = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    echo "Cloned Branch: ${BRANCH_NAME}"
                    echo "Git Tag: ${GIT_TAG}"
                    echo "Git Commit: ${GIT_COMMIT_HASH}"
                }
            }
        }
        stage('Debug Info') {
            steps {
                script {
                    echo "Branch Name: ${BRANCH_NAME}"
                    echo "Git Commit: ${GIT_COMMIT_HASH}"
                    echo "Git Tag: ${GIT_TAG}"
                }
            }
        }
        stage('Building image') {
            steps {
                script {
                    dockerImage = docker.build("${imagename}:${GIT_COMMIT_HASH}")
                }
            }
        }
        stage('Deploy Image') {
            steps {
                script {
                    def tag = ""
                    def additionalTag = ""
                    if (GIT_TAG) {
                        tag = GIT_TAG
                        additionalTag = 'latest'
                    } else if (BRANCH_NAME == 'main') {
                        tag = 'latest'
                    } else if (BRANCH_NAME == 'secondary') {
                        tag = "secondary-${GIT_COMMIT_HASH}"
                    } else {
                        tag = "${BRANCH_NAME}-${GIT_COMMIT_HASH}"
                    }
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push(tag)
                        if (additionalTag) {
                            dockerImage.push(additionalTag)
                        }
                    }
                }
            }
        }
        stage('Remove Unused docker image') {
            steps {
                script {
                    def tag = ""
                    def additionalTag = ""
                    if (GIT_TAG) {
                        tag = GIT_TAG
                        additionalTag = 'latest'
                    } else if (BRANCH_NAME == 'main') {
                        tag = 'latest'
                    } else if (BRANCH_NAME == 'secondary') {
                        tag = "secondary-${GIT_COMMIT_HASH}"
                    } else {
                        tag = "${BRANCH_NAME}-${GIT_COMMIT_HASH}"
                    }
                    sh "docker rmi ${imagename}:${GIT_COMMIT_HASH}"
                    sh "docker rmi ${imagename}:${tag}"
                    if (additionalTag) {
                        sh "docker rmi ${imagename}:${additionalTag}"
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
