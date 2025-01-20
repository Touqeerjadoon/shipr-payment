pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'touqeerjadoon55/shipr-payment'
        DOCKER_IMAGE_CONSUMER = 'touqeerjadoon55/shipr-payment-consumer'
    }
    parameters {
        gitParameter(
            name: 'BRANCH',
            type: 'PT_BRANCH',
            defaultValue: 'develop',
            branchFilter: 'origin/(.*)', 
            selectedValue: 'DEFAULT'
        )
        string(
            name: 'SEMANTIC_VERSION',
            defaultValue: '1.0.0',
            description: 'Enter the semantic version (e.g., 1.0.0) for the Docker image tag.'
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

        // Build and push the main application image
        stage('Build Main Docker Image') {
            steps {
                sh "docker build -t ${env.DOCKER_IMAGE}:${params.SEMANTIC_VERSION} -f Dockerfile ."
            }
        }
        stage('Push Main Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'b4666a98-9a93-47a3-aabd-0439f9dc91fb',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh """
                        echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USER} --password-stdin
                        docker push ${env.DOCKER_IMAGE}:${params.SEMANTIC_VERSION}
                    """
                }
            }
        }

        // Build and push the consumer image
        stage('Build Consumer Docker Image') {
            steps {
                sh "docker build -t ${env.DOCKER_IMAGE_CONSUMER}:${params.SEMANTIC_VERSION} -f Dockerfile.consumer ."
            }
        }
        stage('Push Consumer Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'b4666a98-9a93-47a3-aabd-0439f9dc91fb',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh """
                        echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USER} --password-stdin
                        docker push ${env.DOCKER_IMAGE_CONSUMER}:${params.SEMANTIC_VERSION}
                    """
                }
            }
        }

        // Cleanup
        stage('Cleanup') {
            steps {
                sh """
                    docker rmi ${env.DOCKER_IMAGE}:${params.SEMANTIC_VERSION}
                    docker rmi ${env.DOCKER_IMAGE_CONSUMER}:${params.SEMANTIC_VERSION}
                """
            }
        }
    }
    post {
        success {
            slackSend(
                channel: '#jenkins',
                message: "✅ Pipeline SUCCESSFUL: ${env.JOB_NAME} - ${env.BUILD_NUMBER}\nBranch: ${params.BRANCH}\nVersion: ${params.SEMANTIC_VERSION}\nTriggered by: ${currentBuild.getBuildCauses()[0]?.userName ?: 'Unknown User'}\nMore info: ${env.BUILD_URL}"
            )
        }
        failure {
            slackSend(
                channel: '#jenkins',
                message: "❌ Pipeline FAILED: ${env.JOB_NAME} - ${env.BUILD_NUMBER}\nBranch: ${params.BRANCH}\nVersion: ${params.SEMANTIC_VERSION}\nTriggered by: ${currentBuild.getBuildCauses()[0]?.userName ?: 'Unknown User'}\nMore info: ${env.BUILD_URL}"
            )
        }
    }
}
