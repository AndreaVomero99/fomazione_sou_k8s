pipeline {
    environment {
        imagename = "andreavomero99/ciao"
        registryCredential = 'DockerHub'
        dockerImage = ''
        BRANCH_NAME = ''
        GIT_TAG = ''
        LATEST_TAG = ''
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
                    checkout scm
                    // Ottieni il nome del branch
                    env.BRANCH_NAME = env.BRANCH_NAME
                    // Ottieni il tag Git più recente
                    env.GIT_TAG = sh(script: 'git describe --tags --abbrev=0 || echo ""', returnStdout: true).trim()
                    // Ottieni il commit del tag più recente
                    env.LATEST_TAG = sh(script: "git rev-list -n 1 ${env.GIT_TAG} || echo ''", returnStdout: true).trim()
                    echo "Cloned Branch: ${env.BRANCH_NAME}"
                    echo "Git Tag: ${env.GIT_TAG}"
                    echo "Latest Tag Commit: ${env.LATEST_TAG}"
                }
            }
        }
        stage('Debug Info') {
            steps {
                script {
                    echo "Branch Name: ${env.BRANCH_NAME}"
                    echo "Git Commit: ${env.GIT_COMMIT}"
                    echo "Git Tag: ${env.GIT_TAG}"
                    echo "Latest Tag Commit: ${env.LATEST_TAG}"
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
                    def additionalTag = ""
                    if (env.GIT_TAG && env.GIT_COMMIT == env.LATEST_TAG) {
                        tag = env.GIT_TAG
                        additionalTag = 'latest'
                    } else if (env.BRANCH_NAME == 'main') {
                        tag = 'latest'
                    } else if (env.BRANCH_NAME == 'secondary') {
                        tag = "secondary-${env.GIT_COMMIT}"
                    } else {
                        tag = "${env.BRANCH_NAME}-${env.GIT_COMMIT}"
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
                    if (env.GIT_TAG && env.GIT_COMMIT == env.LATEST_TAG) {
                        tag = env.GIT_TAG
                        additionalTag = 'latest'
                    } else if (env.BRANCH_NAME == 'main') {
                        tag = 'latest'
                    } else if (env.BRANCH_NAME == 'secondary') {
                        tag = "secondary-${env.GIT_COMMIT}"
                    } else {
                        tag = "${env.BRANCH_NAME}-${env.GIT_COMMIT}"
                    }
                    sh "docker rmi ${imagename}:${env.GIT_COMMIT}"
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

