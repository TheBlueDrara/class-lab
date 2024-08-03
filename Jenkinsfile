pipeline {
    agent {
        kubernetes {
            label 'ez-joy-friends'
            idleMinutes 5
            yamlFile 'build-pod.yaml'
            defaultContainer 'ez-docker-helm-build'
        }
    }

    environment {
        GITHUB_URL = 'https://github.com/your-username/your-repo.git'
        IMAGE_NAME = 'thebluedrara/wikiapp'
        GITHUB_CREDENTIALS_ID = 'github-cred'
        DOCKER_HUB_CREDENTIALS_ID = 'docker-cred'
        GITHUB_API_URL = 'https://api.github.com/repos/your-username/your-repo/pulls'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: "${GITHUB_URL}", credentialsId: "${GITHUB_CREDENTIALS_ID}"
            }
        }

        stage('Build Image') {
            steps {
                script {
                    dockerImage = docker.build("${IMAGE_NAME}:${env.BRANCH_NAME}-${env.BUILD_ID}", "--no-cache .")
                }
            }
        }

        stage('Create Pull Request') {
            when {
                not {
                    branch 'main'
                }
            }
            steps {
                script {
                    withCredentials([string(credentialsId: 'github-api-token', variable: 'GITHUB_API_TOKEN')]) {
                        sh """
                        curl -s --header "Authorization: token ${GITHUB_API_TOKEN}" -X POST "${GITHUB_API_URL}" \
                        --data '{
                            "title": "PR from ${env.BRANCH_NAME} into main",
                            "head": "${env.BRANCH_NAME}",
                            "base": "main",
                            "body": "Automatically generated PR",
                            "maintainer_can_modify": true
                        }'
                        """
                    }
                }
            }
        }

        stage('Push Docker Image') {
            when {
                branch 'main'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-cred') {
                        dockerImage.push("latest")
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
