pipeline {
    agent any
    environment {
        FAILED_STAGE = "Unknown"
    }
    stages {
        stage('Checkout') {
            steps {
                script { env.FAILED_STAGE = env.STAGE_NAME }
                checkout scm
            }
        }
        stage('Install Dependencies') {
            steps {
                script { env.FAILED_STAGE = env.STAGE_NAME }
                sh 'python3 -m pip install --break-system-packages -r requirements.txt'
            }
        }
        stage('Run Unit Tests') {
            steps {
                script { env.FAILED_STAGE = env.STAGE_NAME }
                sh 'python3 -m pytest'
            }
        }
        stage('Docker Build') {
            steps {
                script { env.FAILED_STAGE = env.STAGE_NAME }
                sh 'docker build -t sample-target-app:${BUILD_NUMBER} .'
            }
        }
    }
    post {
        failure {
            script {
                def commitHash = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                def jsonPayload = """
                {
                    "job_name": "${env.JOB_NAME}",
                    "build_number": ${env.BUILD_NUMBER},
                    "failed_stage": "${env.FAILED_STAGE}",
                    "commit_hash": "${commitHash}"
                }
                """
                sh """
                curl -X POST http://host.docker.internal:5000/api/webhook/jenkins \
                     -H "Content-Type: application/json" \
                     -d '${jsonPayload}'
                """
            }
        }
    }
}