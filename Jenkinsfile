pipeline {
    agent any

    parameters {
        string(name: 'REPO_URL', defaultValue: '', description: 'URL repo untuk di-clone')
        string(name: 'BRANCH', defaultValue: '', description: 'Branch repo yang ingin digunakan')
        string(name: 'NAMESERVICE', defaultValue: '', description: 'Nama service (untuk deployment)')
        string(name: 'NAMESPACE', defaultValue: '', description: 'Namespace untuk deployment')
        string(name: 'IMAGE', defaultValue: '', description: 'Nama image Docker')
        string(name: 'ARTEFACTS', defaultValue: '', description: 'URL atau lokasi Docker registry')
    }

    environment {
        REMOTE_HOST = '34.1.196.45'
        REMOTE_USER = 'ubuntu'
        REMOTE_IDENTITY_FILE = '/var/lib/jenkins/.ssh/id_rsa'
    }

    stages {
        stage('Clone the repo') { 
            steps {
                script {
                    checkout scmGit(branches: [[name: "origin/${params.BRANCH}"]], userRemoteConfigs: [[url: "${params.REPO_URL}", credentialsId: 'githubbythentoken']])
                }
            }
        }

        stage('Build Image') {
            steps {
                script {
                    def commit = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    sh "docker build --build-arg SSH_KEY='${SSH_KEY}' -t ${params.ARTEFACTS}/${params.IMAGE}:${commit} ."
                }
            }
        }

        stage('Push Docker image') {
            steps {
                script {
                    def commit = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    sh "gcloud auth configure-docker ${params.ARTEFACTS} --quiet"
                    sh "docker push ${params.ARTEFACTS}/${params.IMAGE}:${commit}"
                    sh "docker rmi ${params.ARTEFACTS}/${params.IMAGE}:${commit}"
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
