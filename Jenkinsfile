@Library('Shared') _
def config = securityConfig("devharep2/flask-cicd:${BUILD_NUMBER}", 'flask-cicd-container')

pipeline {
    agent { label 'ci-cd' }

    environment {
        DOCKER_IMAGE = "${config.DOCKER_IMAGE}"
        DOCKER_CREDENTIALS_ID = "${config.DOCKER_CREDENTIALS_ID}"
        CONTAINER_NAME = "${config.CONTAINER_NAME}"
        PORT_MAPPING = "80:5000"
    }

    stages {

        stage('Python Dependency Install') {
            agent {
                docker {
                    image 'python:3.13-slim'
                    args '-u root' // optional, if permissions needed
                }
            }
            steps {
                script {
                    installPythonDepsVm()
                }
            }
        }

        stage('Security Scans') {
            steps {
                script {
                    securityScan()
                }
            }
        }

        stage('Unit Test') {
            agent {
                docker {
                    image 'python:3.13-slim'
                    args '-u root'
                }
            }
            steps {
                script {
                    unitTest()
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build(DOCKER_IMAGE)
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    pushDockerImage(DOCKER_IMAGE, DOCKER_CREDENTIALS_ID)
                }
            }
        }

        stage('Deploy On Deploying') {
            steps {
                sshagent(credentials: ['ssh-ec2']) {
                    script {
                        remoteDockerDeploy(
                            DOCKER_IMAGE,
                            CONTAINER_NAME,
                            PORT_MAPPING,
                            'ssh-ec2'
                        )
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                sendSuccessEmailNotification(env.FLASK_EMAIL_RECIPIENTS)
            }
        }
        failure {
            script {
                sendFailureEmailNotification(env.FLASK_EMAIL_RECIPIENTS)
            }
        }
    }
}
