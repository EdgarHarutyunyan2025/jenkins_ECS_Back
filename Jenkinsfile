pipeline {
    agent any

    environment {
        AWS_REGION = 'eu-central-1'
        IMAGE_TAG = "backend-pr-${env.BUILD_NUMBER}"
	TELEGRAM_BOT_TOKEN = "7072041087:AAFEJ4XOyur7rBUJopRGqiBFGUAKQBBvNBo"
	TELEGRAM_CHAT_ID   = 6912904630 
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('backend_image') {
                    sh 'docker build -t $IMAGE_TAG .'
                }
            }
        }

        stage('Configure AWS') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds-id'
                ]]) {
                    sh 'aws configure set region $AWS_REGION'
                }
            }
        }

        stage('Terraform Init & Plan') {
            steps {
                script {
                    sh 'terraform init'
                    def status = sh(
                        script: "terraform plan -no-color -var='docker_image=${IMAGE_TAG}' > backend-plan.txt",
                        returnStatus: true
                    )
                    currentBuild.result = (status == 0) ? 'SUCCESS' : 'FAILURE'
                }
            }
        }

        stage('Send Telegram') {
            steps {
                script {
                    def message = (currentBuild.result == 'SUCCESS') ?
                        "✅ Terraform plan successful (Build #${env.BUILD_NUMBER})" :
                        "❌ Terraform plan failed (Build #${env.BUILD_NUMBER})"

                    sh """
                        curl -s -X POST https://api.telegram.org/bot${env.TELEGRAM_BOT_TOKEN}/sendMessage \\
                        -d chat_id=${env.TELEGRAM_CHAT_ID} \\
                        -d text="${message}"
                    """
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'backend-plan.txt', onlyIfSuccessful: true
        }
    }
}
