pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'weather-app'
        DOCKER_TAG = 'latest'
        DOCKER_USERNAME = 'aditya7652' 
        DOCKER_REGISTRY = 'docker.io'
        GITHUB_REPO = 'https://github.com/aditya-7562/Weather-Project.git'
        GITHUB_TOKEN = credentials('8e28fd18-016f-44bc-99b2-f2bf65e51f3c')
        DOCKER_PASSWORD = credentials('docker-hub-password')
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: "$GITHUB_REPO"
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $DOCKER_USERNAME/$DOCKER_IMAGE:$DOCKER_TAG .'
                }
            }
        }
        stage('Run Docker Container') {
            steps {
                script {
                    sh 'docker run -d -p 8080:80 --name weather-app-container $DOCKER_USERNAME/$DOCKER_IMAGE:$DOCKER_TAG'
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                    sh 'docker push $DOCKER_USERNAME/$DOCKER_IMAGE:$DOCKER_TAG'
                }
            }
        }
        stage('Deploy to GitHub Pages') {
            steps {
                script {
                    sh '''
                        git config --global user.email "aditya.12114424@lpu.in"
                        git config --global user.name "aditya-7562"

                        # Clone the gh-pages branch securely using GitHub token
                        git clone --depth=1 --branch=gh-pages https://x-access-token:$GITHUB_TOKEN@github.com/aditya-7562/Weather-Project.git gh-pages
                        cd gh-pages

                        # Remove old files to ensure a clean deployment
                        rm -rf *

                        # Copy only necessary static files
                        cp -r ../* .

                        # Remove unnecessary files from the deployment
                        rm -rf .gitignore Dockerfile Jenkinsfile README.md

                        # Commit and push changes
                        git add .
                        git commit -m "Deploy updated weather app to GitHub Pages"
                        git push origin gh-pages
                    '''
                }
            }
}

    }
    post {
        always {
            script {
                sh '''
                    docker ps -q --filter "name=weather-app-container" | xargs -r docker stop
                    docker ps -a -q --filter "name=weather-app-container" | xargs -r docker rm
                '''
            }
        }
    }
}
