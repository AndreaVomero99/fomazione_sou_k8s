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
                    // Checkout the branch or tag that triggered the build
                    checkout scm
                    // Get the current branch name
                    BRANCH_NAME = sh(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()
                    // Get the tag if it exists
                    GIT_TAG = sh(script: 'git describe --tags --exact-match 2>/dev/null || echo ""', returnStdout: true).trim()
                    // Get the short commit hash
                    GIT_COMMIT_HASH = sh(script: 'git rev-parse --short=7 HEAD', returnStdout: true).trim()
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
                    def tag = ""
                    if (GIT_TAG) {
                        tag = GIT_TAG
                    } else if (BRANCH_NAME == 'main') {
                        tag = 'latest'
                    } else if (BRANCH_NAME == 'secondary') {
                        tag = "secondary-${GIT_COMMIT_HASH}"
                    } else {
                        tag = "${BRANCH_NAME}-${GIT_COMMIT_HASH}"
                    }
                    // Build the Docker image with the determined tag
                    dockerImage = docker.build("${imagename}:${tag}")
                }
            }
        }
        stage('Deploy Image') {
            steps {
                script {
                    docker.withRegistry('', registryCredential) {
                        // Push the Docker image with the determined tag
                        dockerImage.push()
                        // If the build was triggered by a git tag, also push the image with the 'latest' tag
                        if (GIT_TAG) {
                            dockerImage.push('latest')
                        }
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
                    } else if (BRANCH_NAME == 'main') {
                        tag = 'latest'
                    } else if (BRANCH_NAME == 'secondary') {
                        tag = "secondary-${GIT_COMMIT_HASH}"
                    } else {
                        tag = "${BRANCH_NAME}-${GIT_COMMIT_HASH}"
                    }
                    // Remove the Docker images to free up space
                    sh "docker rmi ${imagename}:${tag}"
                    if (GIT_TAG) {
                        sh "docker rmi ${imagename}:latest"
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
