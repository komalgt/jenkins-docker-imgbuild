pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'yourdockerhubusername/your-app:${BUILD_NUMBER}'
        DOCKER_HUB_CREDS = credentials('dockerhub-creds')
        GITHUB_TOKEN = credentials('github-token')
        GITHUB_REPO = 'yourgithubusername/your-app'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
                        sh "docker build -t ${DOCKER_IMAGE} ."
                    }
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
                        sh "docker push ${DOCKER_IMAGE}"
                    }
                }
            }
        }
        stage('Create GitHub Release Tag') {
            steps {
                script {
                    def TAG_NAME = "v${BUILD_NUMBER}"
                    def json = """{
                        "tag_name": "${TAG_NAME}",
                        "target_commitish": "main",
                        "name": "${TAG_NAME}",
                        "body": "Automatic release by Jenkins",
                        "draft": false,
                        "prerelease": false
                    }"""
                    sh """
                    curl -s -H "Authorization: token ${GITHUB_TOKEN}" \\
                        -d '${json}' \\
                        https://api.github.com/repos/${GITHUB_REPO}/releases
                    """
                }
            }
        }
    }
}
