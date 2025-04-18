pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'weather-app'
        DOCKER_TAG = 'latest'
        DOCKER_USERNAME = 'aditya7562'
        DOCKER_REGISTRY = 'docker.io'
        GITHUB_REPO = 'https://github.com/aditya-7562/Weather-Project.git'
        GITHUB_TOKEN = credentials('GITHUB_TOKEN') // Jenkins secret
    }

    options {
        disableConcurrentBuilds()
        skipDefaultCheckout()
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: "${GITHUB_REPO}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_USERNAME}/${DOCKER_IMAGE}:${DOCKER_TAG} .'
            }
        }

        stage('Run Docker Container') {
            steps {
                sh 'docker run -d -p 8080:80 --name weather-app-container ${DOCKER_USERNAME}/${DOCKER_IMAGE}:${DOCKER_TAG}'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-password', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''
                        echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                        docker push ${DOCKER_USERNAME}/${DOCKER_IMAGE}:${DOCKER_TAG}
                    '''
                }
            }
        }

        stage('Deploy to GitHub Pages') {
            steps {
                sh '''
                    git config --global user.email "aditya.12114424@lpu.in"
                    git config --global user.name "aditya-7562"

                    git clone --depth=1 --branch=gh-pages https://x-access-token:$GITHUB_TOKEN@github.com/aditya-7562/Weather-Project.git gh-pages
                    cd gh-pages

                    rm -rf *

                    cp -r ../* .

                    rm -rf .gitignore Dockerfile Jenkinsfile README.md

                    git add .
                    git commit -m "Deploy updated weather app to GitHub Pages"
                    git push origin gh-pages
                '''
            }
        }
    }

    post {
        always {
            sh '''
                docker ps -q --filter "name=weather-app-container" | xargs -r docker stop || true
                docker ps -a -q --filter "name=weather-app-container" | xargs -r docker rm || true
            '''
        }
        success {
            echo '✅ Build completed successfully!'
        }
        failure {
            echo '❌ Build failed! Please check the logs.'
        }
    }
}

