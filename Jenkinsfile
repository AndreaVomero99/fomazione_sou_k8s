pipeline {
    environment {
        imagename = "andreavomero99/ciao"
        registryCredential = 'DockerHub'
        dockerImage = ''
        BRANCH_NAME = "${env.GIT.LOCAL_BRANCH}"
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
                    git credentialsId: 'GitHub', url: 'https://github.com/AndreaVomero99/fomazione_sou_k8s'
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
                    def additionalTag = ""
                    if (GIT_TAG) {
                        tag = GIT_TAG
                        additionalTag = 'latest'
                    } else if (BRANCH_NAME == 'main') {
                        tag = 'latest'
                    } else if (BRANCH_NAME == 'secondary') {
                        tag = "secondary-${env.GIT_COMMIT}"
                    } else {
                        tag = "${BRANCH_NAME}-${env.GIT_COMMIT}"
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
                        tag = "secondary-${env.GIT_COMMIT}"
                    } else {
                        tag = "${BRANCH_NAME}-${env.GIT_COMMIT}"
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
