pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'touqeerjadoon55/shipr-payment'
    }
    parameters {
        gitParameter(
            name: 'BRANCH',
            type: 'PT_BRANCH',
            defaultValue: 'develop',
            branchFilter: 'origin/(.*)', 
            selectedValue: 'DEFAULT'
        )
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: "${params.BRANCH}",
                    credentialsId: '240a9f71-d6eb-4bee-af7b-1b6e106f2d18',
                    url: 'https://github.com/Touqeerjadoon/shipr-payment.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${env.DOCKER_IMAGE}:${params.BRANCH} ."
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'b4666a98-9a93-47a3-aabd-0439f9dc91fb',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh """
                        echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USER} --password-stdin
                        docker push ${env.DOCKER_IMAGE}:${params.BRANCH}
                    """
                }
            }
        }
        stage('Cleanup') {
            steps {
                sh "docker rmi ${env.DOCKER_IMAGE}:${params.BRANCH}"
            }
        }
    }
    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
