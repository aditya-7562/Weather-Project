pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'weather-app'
        DOCKER_TAG = 'latest'
        DOCKER_USERNAME = 'aditya7652' 
        DOCKER_REGISTRY = 'docker.io'
        GITHUB_REPO = 'https://github.com/aditya-7562/Weather-Project.git'
        GITHUB_TOKEN = credentials('GITHUB_TOKEN') // GitHub Token
        DOCKER_PASSWORD = credentials('docker-hub-password') // Docker Hub credentials
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
                    // Build the Docker image
                    sh 'docker build -t $DOCKER_USERNAME/$DOCKER_IMAGE:$DOCKER_TAG .'
                }
            }
        }
        stage('Run Docker Container') {
            steps {
                script {
                    // Run the container (detached)
                    sh 'docker run -d -p 8080:80 --name weather-app-container $DOCKER_USERNAME/$DOCKER_IMAGE:$DOCKER_TAG'
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    // Login to DockerHub using credentials
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-password', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                        sh 'docker push $DOCKER_USERNAME/$DOCKER_IMAGE:$DOCKER_TAG'
                    }
                }
            }
        }
        stage('Deploy to GitHub Pages') {
            steps {
                script {
                    // Git config for user
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
        node {
            script {
                // Clean up the container after deployment
                sh '''
                    docker ps -q --filter "name=weather-app-container" | xargs -r docker stop
                    docker ps -a -q --filter "name=weather-app-container" | xargs -r docker rm
                '''
            }
        }
    }
    success {
        echo 'Build completed successfully! 🚀'
    }
    failure {
        echo 'Build failed! Please check the logs for errors. ⚠️'
    }
}
}
